# Enable policy enforcement with Scan 2.0 in a supply chain

This topic tells you how to enable policy enforcement in a supply chain with Supply Chain Security
Tools (SCST) - Scan 2.0.

Policy enforcement is not built into SCST - Scan 2.0 and is not in any of the Out of the Box supply
chains. You must [author a custom supply chain](../scc/authoring-supply-chains.hbs.md) to enable
policy enforcement.

You can enforce a policy in a supply chain by creating a `ClusterImageTemplate` that uses a Tekton
`TaskRun` to evaluate the vulnerabilities and enforce the set policy. This topic provides a sample
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

1. Create a secret `metadata-store-access-token` in the developer namespace:

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

1. Create a secret `metadata-store-cert` in the developer namespace:

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

## <a id="task-run-sample"></a> Edit the `TaskRun` sample to enforce the policy

The following sample `TaskRun` waits until the vulnerability data is available for the image. When
the data is available, the vulnerabilities are aggregated by severity. The policy that `GATE` sets
determines whether `TaskRun` succeeds or fails.

Edit the sample for your needs:

```yaml
apiVersion: tekton.dev/v1
kind: TaskRun
metadata:
  name: enforce-policy
spec:
  serviceAccountName: SERVICE-ACCOUNT-THAT-CAN-READ-TAP-IMAGES # A service account in the developer namespace that can read Tanzu Application Platform images.
  taskSpec:
    steps:
    - name: enforce-policy
      image: TASK-RUN-IMAGE-WITH-CURL-AND-JQ # A TaskRun image that contains curl and jq
      script: |
        if [ ${GATE} -eq "none" ]; then
            exit 0
        fi

        IMAGE_DIGEST=$(echo ${IMAGE} | cut -d "@" -f 2)
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
        value: "critical" # Policy enforcement gate
      - name: METADATA-STORE-URL-NAME
        value: METADATA-STORE-URL-VALUE
      - name: IMAGE
        value: IMAGE-WITH-DIGEST-THAT-WAS-SCANNED # Image that was scanned and will be templated
      - name: ACCESS_TOKEN # The access token from the secret is set as an environment variable
        valueFrom:
          secretKeyRef:
            name: METADATA-STORE-ACCESS-TOKEN
            key: accessToken
      volumeMounts:
      - name: cert
        mountPath: /var/cert
    volumes:
    - name: SECRET-CONTAINING-METADATA-STORE-CERT # The certificate from the Metadata Store is mounted as a file to the TaskRun
      secret:
        secretName: metadata-store-cert
```

Where:

- `GATE` is an environment variable that sets the threshold for severity in the policy. The accepted
  values are `low`, `medium`, `high`, and `critical`. For example, if the `GATE` is set to `high`
  then the `TaskRun` fails if it finds `high` or `critical` vulnerabilities for the image. If the
  `GATE` is set to `none`, no policy is enforced.

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

1. Embed the updated `TaskRun` in a `ClusterImageTemplate` by setting the values for
   environment variables and properties you used when
   [editing the TaskRun sample to enforce the policy](#task-run-sample):

    ```yaml
    #@ load("@ytt:data", "data")
    ---
    apiVersion: carto.run/v1alpha1
    kind: ClusterImageTemplate
    metadata:
      name: scan-policy-template
    spec:
      imagePath: .status.compliantArtifact.registry.image
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

1. Plug in the created `ClusterImageTemplate` after the scan step in the newly created supply chain.
   The following example shows where to plug in the policy template in the supply chain.

    ```yaml
    ---
    apiVersion: carto.run/v1alpha1
    kind: ClusterSupplyChain
    metadata:
      name: CUSTOM-SUPPLY-CHAIN-NAME
    spec:
      .... # previous steps
      resources:
      - name: image-scanner # <----------- scan step in the supply chain
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

      - name: policy # <------------------- insert policy cluster image template after scanner
        templateRef:
          kind: ClusterImageTemplate
          name: scan-policy-template
        images:
        - resource: image-scanner
          name: image
        ... # supply chain continues
    ```

1. Create a workload in the developer namespace to trigger the supply chain:

    ```yaml
    apiVersion: carto.run/v1alpha1
    kind: Workload
    metadata:
      labels:
        app.kubernetes.io/part-of: test-policy
        apps.tanzu.vmware.com/workload-type: web
      name: test-policy
      namespace: app-scanning
    spec:
      image: image@sha256:digest
    ```
