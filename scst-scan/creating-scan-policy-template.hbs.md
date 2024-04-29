# Enable policy enforcement with Scan 2.0 in a supply chain

This topic tells you how to enable policy enforcement in a supply chain with Supply Chain Security
Tools (SCST) - Scan 2.0.

Policy enforcement is **not** inbuilt in SCST - Scan 2.0 and is **not** included in any of the
out of the box supply chains. A custom supply chain needs to be created to enable policy enforcement.
Follow steps [here](../scc/authoring-supply-chains.hbs.md) to author a new supply chain.

A policy can be enforced in a supply chain by creating a `ClusterImageTemplate` that stamps out a
Tekton `TaskRun` that evaluates the vulnerabilities and enforces the policy set. A sample of
the task run and cluster image template are provided in this topic.

## Prerequisites for the task run

The `TaskRun` queries the metadata store to get the list of vulnerabilities for the image.
To authenticate with the MDS (Metadata Store) API, an accessToken and certificate are needed.
Complete the following steps:

1. Build an image that contains `curl` and `jq`. This image is used by the task run.

1. Get the access token from MDS:

    ```console
    ACCESS-TOKEN=$(kubectl get secrets -n metadata-store  metadata-store-read-write-client -o yaml | yq .data.token | base64 -d)
    ```

1. Create a secret `metadata-store-access-token` in the developer namespace.

    ```yaml
    kind: Secret
    apiVersion: v1
    metadata:
      name: metadata-store-access-token
    stringData:
      accessToken: <ACCESS-TOKEN>
    ```

1. Get the CA certificate from MDS:

    ```console
    MDS_CA_CERT=$(kubectl get secret -n metadata-store ingress-cert -o json | jq -r ".data.\"ca.crt\"" | base64 -d)
    ```

1. Create a secret `metadata-store-cert` in the developer namespace:

    ```yaml
    kind: Secret
    apiVersion: v1
    metadata:
      name: metadata-store-cert
    stringData:
      caCrt: |
        <MDS_CA_CERT>
    ```

## Task run sample that enforces policy

The _sample_ task run provided below waits until the vulnerability data is available for the image.
When the data is available, the vulnerabilities are aggregated by severity. The task run succeeds or
fails based on the policy `GATE` set.

The sample task run has a few environment variables/properties that need to updated:

- GATE: _Environment variable_ Variable that sets the threshold for severity in the policy.
Accepted values: low, medium, high or critical
For example: If the `GATE` is set to `high`, the task run fails if it finds high or critical
vulnerabilities for the image.
If the `GATE` is set to `none`, no policy is enforced.

- METADATA_STORE_URL: _Environment variable_ The url to reach metadata store.
In a single cluster deployment, the value would be `metadata-store-app.metadata-store.svc.cluster.local:8443`.
In a multi cluster deployment, the value would be `metadata-store.<view-cluster-ingress-domain>`.

- IMAGE: _Environment variable_ The image that was built and scanned in the
previous steps of the supply chain. This should substituted using template `#@ data.values.image`
while embedding the task run in the `ClusterImageTemplate`. Templating this value will allow it be
passed through the supply chain.

- spec.serviceAccountName: _Property_ This service account in the developer namespace should have
access to read TAP images. This will be used by the TaskRun to pull images for its execution steps.

- spec.taskSpec.steps[_].image: _Property_ Any image that contains `curl` and `jq` commands.

- spec.taskSpec.steps[_].env[_].name['ACCESS_TOKEN'].valueFrom.secretKeyRef.name: _Property_ Name of
the secret that contains the MDS read access token.

- spec.taskSpec.volumes[_].name: _Property_ Name of the secret that contains the MDS cert.

>**Note:** The below task run is a sample. It needs to updated while embedding in `ClusterImageTemplate`

```yaml
apiVersion: tekton.dev/v1
kind: TaskRun
metadata:
  name: enforce-policy
spec:
  serviceAccountName: <sa-that-can-read-tap-images> # A service account in the developer namespace that can read TAP images.
  taskSpec:  
    steps:
    - name: enforce-policy
      image: <task-run-image-with-curl-and-jq> # A task run image that contains curl and jq
      script: |
        if [ ${GATE} -eq "none" ]; then
            exit 0
        fi

        IMAGE_DIGEST=$(echo ${IMAGE} | cut -d "@" -f 2)
        while true; do
          response_code=$(curl https://$METADATA_STORE_URL/api/v1/images/${IMAGE_DIGEST} -H "Authorization: Bearer ${ACCESS_TOKEN}" -H 'accept: application/json' --cacert /var/cert/caCrt -o vulnerabilities.json -w "%{http_code}")
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
      - name: METADATA_STORE_URL
        value: <metadata-store-url>
      - name: IMAGE
        value: <image-with-digest-that-was-scanned> # Image that was scanned. Should be templated
      - name: ACCESS_TOKEN # Access token from the secret is set as an environment variable
        valueFrom:
          secretKeyRef:
            name: metadata-store-access-token
            key: accessToken
      volumeMounts:
      - name: cert
        mountPath: /var/cert
    volumes:
    - name: cert # Cert from MDS is mounted as a file to the task run
      secret:
        secretName: metadata-store-cert
```

## Include the policy ClusterImageTemplate in the newly authored supply chain

1. Embed the updated task run in a `ClusterImageTemplate`. Ensure to set correct/desired values for
environment variables and properties mentioned in the previous [step](#task-run-sample-that-enforces-policy).

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

Below example shows where to plug in the policy template in the supply chain.

  ```yaml
    ---
    apiVersion: carto.run/v1alpha1
    kind: ClusterSupplyChain
    metadata:
      name: <custom-supply-chain-name>
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
        ... # <supply chain continues>
    ```

1. Create a workload in the developer namespace to trigger the supply chain.

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
