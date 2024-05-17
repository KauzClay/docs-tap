# Create a supply chain that performs a SonarQube scan

This topic tells you how to create a supply chain that performs a SonarQube scan. The SonarQube scan
component currently only supports scanning Maven projects.

## <a id="prerequisites"></a> Prepare

To prepare:

- [Install Tanzu Supply Chain packages](../../../supply-chain/platform-engineering/how-to/installing-supply-chain/install-authoring-profile.hbs.md#tsc-packages)
- [Install the Tanzu CLI](../../../install-tanzu-cli.hbs.md)
- [Install the Tanzu Supply Chain CLI plug-ins](../../../supply-chain/platform-engineering/how-to/install-the-cli.hbs.md)

## <a id="sonarqube-scan"></a> Create a supply chain with the SonarQube Scan component

Create a Supply Chain with the SonarQube Scan component by using the Tanzu Supply Chain CLI plug-in:

1. Initialize Tanzu Supply Chain by running:

   ```console
   tanzu supplychain init --group GROUP-NAME
   ```

   Example:

   ```console
   $ tanzu supplychain init --group example.com
   Initializing group example.com
   Creating directory structure
   ├─ supplychains/
   ├─ components/
   ├─ pipelines/
   ├─ tasks/
   └─ config.yaml

   Writing group configuration to config.yaml
   ```

1. Generate A supply chain by running:

   ```console
   tanzu supplychain generate --kind NAME-OF-SUPPLY-CHAIN \
   --description DESCRIPTION-FOR-SUPPLY-CHAIN \
   --component source-git-provider-1.0.0 \
   --component sonarqube-sast-scan-1.0.0
   ```

   Example output:

   ```console
   $ tanzu supplychain generate --kind SonarQubeSC \
     --description SonarQube \
     --component source-git-provider-1.0.0 \
     --component sonarqube-sast-scan-1.0.0

   ✓ Successfully fetched all component dependencies
   Created file supplychains/sonarqubesc.yaml
   Created file components/sonarqube-sast-scan-1.0.0.yaml
   Created file components/source-git-provider-1.0.0.yaml
   Created file pipelines/sonarqube-sast-scan.yaml
   Created file pipelines/source-git-provider.yaml
   Created file tasks/fetch-tgz-content-oci.yaml
   Created file tasks/sonarqube-maven-scan.yaml
   Created file tasks/source-git-check.yaml
   Created file tasks/source-git-clone.yaml
   Created file tasks/store-content-oci.yaml
   ```

### <a id="apply-supply-chain"></a> Apply Supply Chain

By generating the supply chain, you created the following directory structure:

```console
├─ supplychains/
├─ components/
├─ pipelines/
├─ tasks/
```

Apply these directories to the developer namespace where the workload will run by running:

```console
kubectl apply -R -f components -f supplychains -f tasks -f pipelines -n DEV-NAMESPACE
```

Where `DEV-NAMESPACE` is the namespace where the workload will be.

## <a id="customize-sonarqube-task"></a> Customize a SonarQube scan task

This section describes how to customize a SonarQube scan task to run on a different image.

The default image that the SonarQube scan task runs on is a mirror of the `maven:3.9.6` image. If
this Maven version is not compatible with your project to perform the SonarQube scan, you can
customize the task to use your own image with the Maven version you need.

1. Edit the following line in `tasks/sonarqube-maven-scan.yaml`, which you generated earlier from
   the Tanzu Supply Chain CLI:

    ```yaml
    apiVersion: tekton.dev/v1
    kind: Task
    metadata:
    name: sonarqube-maven-scan
    namespace: sonarqube-catalog
    spec:
    ...
      steps:
        - computeResources: {}
          ...
          # change this image field to point to your image
          image: registry.tanzu.vmware.com/tanzu-application-platform/tap-packages@sha256:a263660b3f2d89af528c9344856121ec26d21e97f09c3f70bf823a3ae5d9c8b1
          name: scan
    ...
    ```

1. Apply the custom task by running:

   ```console
   kubectl apply -f tasks/sonarqube-maven-scan.yaml -n DEV-NAMESPACE
   ```

   Where `DEV-NAMESPACE` is the namespace where the workload will be.