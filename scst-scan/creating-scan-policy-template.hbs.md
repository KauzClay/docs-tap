# Enable policy enforcement with Scan 2.0 in a supply chain

This topic tells you how to enable policy enforcement in a supply chain with Supply Chain Security
Tools (SCST) - Scan 2.0.

Policy enforcement is not built into SCST - Scan 2.0 and is not in any of the Out of the Box supply
chains. You must [author a custom supply chain](../scc/authoring-supply-chains.hbs.md) to enable
policy enforcement.

To enforce a policy in a supply chain you need a `ClusterImageTemplate` that uses a Tekton
`TaskRun` to evaluate the vulnerabilities and enforces the set policy. This topic provides a sample
`TaskRun` and a sample `ClusterImageTemplate` for you to edit.

## <a id="prepare"></a> Prepare for the `TaskRun`

The `TaskRun` queries the Metadata Store to get the list of vulnerabilities for the image.
Authenticate with the Metadata Store API by obtaining an access token and a certificate:

1. Build an image that contains `curl` and `jq` for the `TaskRun` to use.

1. Get the access token from the Metadata Store by running:

   ```console
   ACCESS-TOKEN=$(kubectl get secrets -n metadata-store  metadata-store-read-write-client -o yaml \
   | yq .data.token | base64 -d)
   ```

1. Create the secret `metadata-store-access-token-secret.yaml` and include the access token:

    ```yaml
    kind: Secret
    apiVersion: v1
    metadata:
      name: metadata-store-access-token
    stringData:
      accessToken: ACCESS-TOKEN
    ```

    Where `ACCESS-TOKEN` is the access token

1. Get the Certificate Authority (CA) certificate from the Metadata Store by running:

   ```console
   MDS_CA_CERT=$(kubectl get secret -n metadata-store ingress-cert -o json | jq -r ".data.\"ca.crt\"" \
   | base64 -d)
   ```

1. Create the secret `metadata-store-cert-secret.yaml` and include the certificate:

    ```yaml
    kind: Secret
    apiVersion: v1
    metadata:
      name: metadata-store-cert
    stringData:
      caCrt: |
        METADATA-STORE-CA-CERT
    ```

    Where `METADATA-STORE-CA-CERT` is the Metadata Store CA certificate

1. Apply the created secrets in the developer namespace:

    ```console
    kubectl apply -f metadata-store-access-token-secret.yaml -n DEVELOPER-NAMESPACE
    kubectl apply -f metadata-store-cert-secret.yaml -n DEVELOPER-NAMESPACE
    ```

## <a id="task-run-sample"></a> Edit the `TaskRun` sample to enforce the policy

The following sample `TaskRun` waits until the vulnerability data is available for the image in
the Metadata Store. When the data is available, the vulnerabilities are aggregated by severity.
The policy that `GATE` sets determines whether `TaskRun` succeeds or fails.

Edit the sample for your needs:

```yaml
apiVersion: tekton.dev/v1
kind: TaskRun
metadata:
  generateName: enforce-policy-task-run-
spec:
  serviceAccountName: SERVICE-ACCOUNT-THAT-CAN-READ-TAP-IMAGES
  taskSpec:
    params:
    - name: image
      value: IMAGE-WITH-DIGEST-THAT-WAS-SCANNED
    steps:
    - name: enforce-policy
      image: TASK-RUN-IMAGE-WITH-CURL-AND-JQ
      script: |
        if [ ${GATE} -eq "none" ]; then
            exit 0
        fi

        IMAGE_DIGEST=$(echo $(params.image) | cut -d "@" -f 2)
        while true; do
          response_code=$(curl https://$METADATA-STORE-URL/api/v1/images/${IMAGE_DIGEST} -H "Authorization: Bearer ${ACCESS_TOKEN}" -H 'accept: application/json' --cacert /var/cert/caCrt -o vulnerabilities.json -w "%{http_code}")
          if [ $response_code -eq 200 ]; then
            echo "Vulnerabilities data is available in AMR"
            break
          else
            echo "Vulnerabilities data is not available in AMR, waiting..."
            sleep 10
          fi
        done

        critical=$(cat vulnerabilities.json | jq '[.Packages[].Vulnerabilities[] | select(.Ratings[].Severity | contains("Critical"))] | unique | length')
        high=$(cat vulnerabilities.json | jq '[.Packages[].Vulnerabilities[] | select(.Ratings[].Severity | contains("High"))] | unique | length')
        medium=$(cat vulnerabilities.json | jq '[.Packages[].Vulnerabilities[] | select(.Ratings[].Severity | contains("Medium"))] | unique | length')
        low=$(cat vulnerabilities.json | jq '[.Packages[].Vulnerabilities[] | select(.Ratings[].Severity | contains("Low"))] | unique | length')
        vulnerabilitiesCount=0
        case $GATE in
          low)
            vulnerabilitiesCount=$(($low+$medium+$high+$critical))
            ;;
          medium)
            vulnerabilitiesCount=$(($medium+$high+$critical))
            ;;

          high)
            vulnerabilitiesCount=$(($high+$critical))
            ;;

          critical)
            vulnerabilitiesCount=$(($critical))
            ;;
        esac
        echo "Vulnerabilities: ${vulnerabilitiesCount}"
        if [ ${vulnerabilitiesCount} -gt 0 ];then
          exit 1
        fi
      env:
      - name: GATE
        value: "critical"
      - name: METADATA-STORE-URL-NAME
        value: METADATA-STORE-URL-VALUE
      - name: ACCESS_TOKEN
        valueFrom:
          secretKeyRef:
            name: METADATA-STORE-ACCESS-TOKEN
            key: accessToken
      volumeMounts:
      - name: cert
        mountPath: /var/cert
    volumes:
    - name: SECRET-CONTAINING-METADATA-STORE-CERT
      secret:
        secretName: metadata-store-cert
```

Where:

- `GATE` is an environment variable that sets the threshold for severity in the policy. The accepted
  values are `low`, `medium`, `high`, and `critical`. For example, if the `GATE` is set to `high`
  then the `TaskRun` fails if it finds `high` or `critical` vulnerabilities for the image. If the
  `GATE` is set to `none`, no policy is enforced and `TaskRun` succeeds.

- `METADATA-STORE-URL-VALUE` is the URL for reaching the Metadata Store.
  In a single-cluster deployment, the value is `metadata-store-app.metadata-store.svc.cluster.local:8443`.
  In a multicluster deployment, the value is `metadata-store.VIEW-CLUSTER-INGRESS-DOMAIN`.

- `IMAGE-WITH-DIGEST-THAT-WAS-SCANNED` is the image that was built and scanned in the previous
  steps of the supply chain. The environment variable value is set to `#@ data.values.image` while
  embedding the `TaskRun` in the `ClusterImageTemplate`. Template this value to enable it be passed
  through the supply chain.

- `SERVICE-ACCOUNT-THAT-CAN-READ-TAP-IMAGES` is a service account in the developer namespace that
  has access to read Tanzu Application Platform images. `TaskRun` uses this service account to pull
  images for its execution steps.

- `TASK-RUN-IMAGE-WITH-CURL-AND-JQ` is any image that contains `curl` and `jq` commands.

- `METADATA-STORE-ACCESS-TOKEN` is the secret that contains the Metadata Store read access token.

- `SECRET-CONTAINING-METADATA-STORE-CERT` is the name of the secret that contains the Metadata Store
  certificate.

## <a id="inc-the-policy"></a> Include the policy `ClusterImageTemplate` in the newly authored supply chain

To include the policy `ClusterImageTemplate` in the newly authored supply chain:

1. Create `policy-cluster-image-template.yaml` by embedding the updated `TaskRun` in a `ClusterImageTemplate`.
   Set the values for environment variables and properties you used when
   [editing the TaskRun sample to enforce the policy](#task-run-sample):

    ```yaml
    #@ load("@ytt:data", "data")
    ---
    apiVersion: carto.run/v1alpha1
    kind: ClusterImageTemplate
    metadata:
      name: scan-policy-template
    spec:
      imagePath: spec.params[?(@.name=="image")].value
      healthRule:
        multiMatch:
          healthy:
            matchConditions:
              - type: Succeeded
                status: "True"
          unhealthy:
            matchConditions:
              - type: Succeeded
                status: "False"
      ytt: |
        #@ load("@ytt:data", "data")
        #@ load("@ytt:template", "template")
        ---
        apiVersion: tekton.dev/v1
        kind: TaskRun
        metadata:
          name: enforce-policy
        spec:
            ....
    ```

1. Create a new supply chain named `custom-supply-chain.yaml` by following the steps in
   [Author your supply chains](../scc/authoring-supply-chains.hbs.md).

   The scan step generates the vulnerability data that the policy step queries the Metadata Store
   for. Therefore the policy step must be after the scan step and must take the image produced by
   the scan step as an input image. You can place any step that requires an image as an input after
   the policy step.

    ```yaml
    ---
    apiVersion: carto.run/v1alpha1
    kind: ClusterSupplyChain
    metadata:
      name: CUSTOM-SUPPLY-CHAIN-NAME
    spec:
      resources:
      .... # previous steps
      - name: image-scanner
        templateRef:
          kind: ClusterImageTemplate
          name: image-vulnerability-scan-trivy
        params:
          - name: registry
            default:
              server: dev.registry.tanzu.vmware.com
              repository: tanzu-image-signing/test-policy
        images:
          - resource: image-provider
            name: image

      - name: policy
        templateRef:
          kind: ClusterImageTemplate
          name: scan-policy-template
        images:
        - resource: image-scanner
          name: image
        ... # supply chain continues
    ```

## <a id="apply-generated-yaml"></a> Apply the template, supply chain, and workload

To apply the template, supply chain, and workload:

1. Apply the generated `ClusterImageTemplate` and `ClusterSupplyChain` by running:

    ```console
    kubectl apply -f policy-cluster-image-template.yaml
    kubectl apply -f custom-supply-chain.yaml
    ```

1. Create a workload in the developer namespace that can trigger the supply chain.
   For more information about selectors and how to use them with a custom supply chain, see
   [Providing your own supply chain](../scc/authoring-supply-chains.hbs.md#own-sup-chain).

## <a id="troubleshooting-policy"></a> Troubleshoot the policy

If the policy is still waiting to find the vulnerabilities data for the image in the Metadata Store:

1. Verify that the observer is healthy and running.
1. Verify that the observer registered the controller to monitor the `ImageVulnerabilityScan` kind.
   For more information about troubleshooting the observer, see
   [Troubleshooting Artifact Metadata Repository](../scst-store/amr/troubleshooting.hbs.md).
