# Enable policy enforcement in a supply chain with Scan 2.0

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

1. Generate an image that has the curl and jq commands. You can either:

    - (Recommended) Build your own image.
    - Use a base Ubuntu image. This gets you started faster but is only for testing purposes.

    Build your own image
    : Build an image that contains curl and jq for the tekton `Task` to use.

      VMware recommends that you use this option because the image will have stable versions and dependencies
      that are deterministic. It will not need to download curl and jq each time the script runs.

      To build your image:

      1. Create a `Dockerfile` with the following in a blank directory:

          ```dockerfile
          FROM ubuntu:latest

          RUN apt-get update
          RUN apt-get install -y jq curl
          ```

      1. Build and push the image to a registry that is accessible by the build cluster by running:

          ```console
          docker build . -t REGISTRY-URL-LOCATION/IMAGE-NAME:IMAGE-TAG
          docker push REGISTRY-URL-LOCATION/IMAGE-NAME:IMAGE-TAG
          ```

          Where:

          - `REGISTRY-URL-LOCATION` is the registry URL. For example, `registry.hub.docker.com/project`.
          - `IMAGE-NAME` is the name of the image. For example, `curl-jq-bash`.
          - `IMAGE-TAG` is the tag of the image. For example, `latest`.

      1. If you are pushing to a private registry, run the following command on the Build cluster:

          ```console
          tanzu secret registry add registry-credentials --server REGISTRY-SERVER --username REGISTRY-USERNAME --password REGISTRY-PASSWORD --export-to-all-namespaces --yes --namespace tap-install
          ```

          Where:

          - `REGISTRY-SERVER` is the registry URL. For example, `registry.hub.docker.com`.
          - `REGISTRY-USERNAME` the user name that is allowed to read the pushed curl jq image.
          - `REGISTRY-PASSWORD` the password that is allowed to read the pushed curl jq image.

    Use a base Ubuntu image
    : To get started quicker and for testing purposes, you can embed the downloading of curl and jq
      in the `Task` script and use a base Ubuntu image.

      > **Note** This option is not recommended in production environments. VMware recommends that you
      > build your own image that contains curl and jq so that it has predetermined dependencies and versions.

      For example, in the `Task` YAML:

      ```yaml
      ---
      apiVersion: tekton.dev/v1
      kind: Task
      metadata:
        name: tekton-policy-enforcement
        namespace: DEVELOPER-NAMESPACE
      spec:
        params:
          - name: image
            type: string
        steps:
          - name: enforce-policy
            image: ubuntu:latest # <-- ubuntu base image
            ...
            script: |
              apt-get update # <-- update download links
              apt-get install -y jq curl # <-- install jq and curl

              if [ ${GATE} = "none" ]; then
                  exit 0
              fi
            ...
      ```

1. In the View cluster, get the access token and the Certificate Authority (CA) certificate from
   the Metadata Store by running these commands:

   ```console
   ACCESS_TOKEN=$(kubectl get secrets -n metadata-store  metadata-store-read-write-client -o json \
   | jq -r ".data.token" | base64 -d)
   ```

   ```console
   METADATA_STORE_CA_CERT=$(kubectl get secret -n metadata-store ingress-cert -o json | jq -r ".data.\"ca.crt\"" \
   | base64 -d)
   ```

1. Verify that both variables are populated by running:

   ```console
   echo $ACCESS_TOKEN
   echo $METADATA_STORE_CA_CERT
   ```

1. In the Build cluster, create `metadata-store-access-token` and `metadata-store-cert` secrets by running:

    ```console
    DEVELOPER_NAMESPACE=DEVELOPER-NAMESPACE

    kubectl create secret generic metadata-store-access-token \
      --from-literal=accessToken="${ACCESS_TOKEN}" -n ${DEVELOPER_NAMESPACE}

    kubectl create secret generic metadata-store-cert \
      --from-literal=caCrt="${METADATA_STORE_CA_CERT}" -n ${DEVELOPER_NAMESPACE}
    ```

    Where `DEVELOPER-NAMESPACE` is the developer namespace.

## <a id="task-run-sample"></a> Edit the `Task` sample to enforce the policy

The following sample `Task` waits until the vulnerability data is available for the image in
the Metadata Store. When the data is available, the vulnerabilities are aggregated by severity.
The policy that `GATE` sets determines whether `TaskRun` succeeds or fails.

1. Create a file named `task-policy-enforcement.yaml` using the following template, and edit it for your needs:

    ```yaml
    ---
    apiVersion: tekton.dev/v1
    kind: Task
    metadata:
      name: tekton-policy-enforcement
      namespace: DEVELOPER-NAMESPACE
    spec:
      params:
        - name: image
          type: string
      steps:
        - name: enforce-policy
          image: TASK-RUN-IMAGE-WITH-CURL-AND-JQ
          env:
          - name: GATE
            value: THRESHOLD
          - name: METADATA_STORE_URL
            value: METADATA-STORE-URL
          - name: ACCESS_TOKEN
            valueFrom:
              secretKeyRef:
                name: metadata-store-access-token
                key: accessToken
          script: |
            if [ ${GATE} = "none" ]; then
                exit 0
            fi

            IMAGE_DIGEST=$(echo $(params.image) | cut -d "@" -f 2)
            while true; do
              response_code=$(curl https://${METADATA_STORE_URL}/api/v1/images/${IMAGE_DIGEST} -H "Authorization: Bearer ${ACCESS_TOKEN}" -H 'accept: application/json' --cacert /var/cert/caCrt -o vulnerabilities.json -w "%{http_code}")
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
          volumeMounts:
          - name: cert
            mountPath: /var/cert
      volumes:
        - name: cert
          secret:
            secretName: metadata-store-cert
    ```

    Where:

    - `THRESHOLD` sets the threshold for severity in the policy. The accepted
      values are `low`, `medium`, `high`, and `critical`. For example, if the `THRESHOLD` is set to `high`
      then the `TaskRun` fails if it finds `high` or `critical` vulnerabilities for the image. If the
      `THRESHOLD` is set to `none`, no policy is enforced and `TaskRun` succeeds.

    - `METADATA-STORE-URL-VALUE` is the URL for reaching the Metadata Store.
      The format is `metadata-store.` followed by the View cluster ingress domain, that is, `metadata-store.VIEW-CLUSTER-INGRESS-DOMAIN`.
      Alternatively, you can get the URL from the View cluster by running:

      ```console
      kubectl get httpproxy metadata-store-ingress -n metadata-store -o jsonpath='{.spec.virtualhost.fqdn}'
      ```

    - `TASK-RUN-IMAGE-WITH-CURL-AND-JQ` is any image that contains the bash, curl, and jq commands.
      If you did not build an image in the first step, you can use the `ubuntu:latest` image and
      install jq and curl at the beginning of the script section. For example:

        ```console
        image: ubuntu:latest
        ...
        ...
        script: |
          apt-get update
          apt-get install -y jq curl

          if [ ${GATE} = "none" ]; then
              exit 0
          fi
        ...
        ...
        ```

      > **Note** This is not recommended in production environments. VMware recommends that you
      > build your own image that contains curl and jq with predetermined dependencies and versions.

    - `DEVELOPER-NAMESPACE` is the developer namespace.

1. Apply the Task to the `DEVELOPER-NAMESPACE` by running:

    ```console
    kubectl apply -f task-policy-enforcement.yaml -n DEVELOPER-NAMESPACE
    ```

    Where `DEVELOPER-NAMESPACE` is the developer namespace.

    Apply this to every developer namespace that uses the policy enforcement.

## <a id="inc-the-policy"></a> Include the policy `ClusterImageTemplate` in the newly authored supply chain

To include the policy `ClusterImageTemplate` in the newly authored supply chain:

1. Create a file named `policy-cluster-image-and-runnable-template.yaml` as follows:

    ```yaml
    apiVersion: carto.run/v1alpha1
    kind: ClusterImageTemplate
    metadata:
      name: policy-enforcement-template
    spec:
      healthRule:
        singleConditionType: Ready
      imagePath: .status.outputs.imagePath
      lifecycle: mutable
      ytt: |-
        #@ load("@ytt:data", "data")

        #@ def merge_labels(fixed_values):
        #@   labels = {}
        #@   if hasattr(data.values.workload.metadata, "labels"):
        #@     exclusions = ["kapp.k14s.io/app", "kapp.k14s.io/association"]
        #@     for k,v in dict(data.values.workload.metadata.labels).items():
        #@       if k not in exclusions:
        #@         labels[k] = v
        #@       end
        #@     end
        #@   end
        #@   labels.update(fixed_values)
        #@   return labels
        #@ end

        #@ def merged_tekton_params():
        #@   params = []
        #@   params.append({ "name": "image", "value": data.values.image })
        #@   return params
        #@ end
        ---
        apiVersion: carto.run/v1alpha1
        kind: Runnable
        metadata:
          name: #@ data.values.workload.metadata.name + "-policy"
          labels: #@ merge_labels({ "app.kubernetes.io/component": "test" })
        spec:
          retentionPolicy:
            maxFailedRuns: 1
            maxSuccessfulRuns: 1

          #@ if/end hasattr(data.values.workload.spec, "serviceAccountName"):
          serviceAccountName: #@ data.values.workload.spec.serviceAccountName

          runTemplateRef:
            name: tekton-policy-enforcement-taskrun
            kind: ClusterRunTemplate

          resource:
            apiVersion: tekton.dev/v1
            kind: TaskRun

          inputs:
            tekton-params: #@ merged_tekton_params()

    ---
    apiVersion: carto.run/v1alpha1
    kind: ClusterRunTemplate
    metadata:
      name: tekton-policy-enforcement-taskrun
    spec:
      outputs:
        imagePath: spec.params[?(@.name=="image")].value
      template:
        apiVersion: tekton.dev/v1
        kind: TaskRun
        metadata:
          generateName: $(runnable.metadata.name)$-
          labels: $(runnable.metadata.labels)$
        spec:
          params: $(runnable.spec.inputs.tekton-params)$
          serviceAccountName: default
          taskRef:
            name: tekton-policy-enforcement
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
          name: policy-enforcement-template
        images:
        - name: image
          resource: image-scanner

      - name: config-provider
        params:
        - default: default
          name: serviceAccount
        templateRef:
          kind: ClusterConfigTemplate
          name: convention-template
        images:
        - name: image
          resource: policy

      .... # supply chain continues
    ```

    Where `CUSTOM-SUPPLY-CHAIN-NAME` is the name of the custom supply chain.

## <a id="apply-generated-yaml"></a> Apply the template, supply chain, and workload

To apply the template, supply chain, and workload:

1. Apply the generated `ClusterImageTemplate` and `ClusterSupplyChain` by running:

    ```console
    kubectl apply -f policy-cluster-image-and-runnable-template.yaml
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
