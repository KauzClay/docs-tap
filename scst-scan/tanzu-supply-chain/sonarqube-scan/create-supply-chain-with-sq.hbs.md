# Create a supply chain that performs a SonarQube scan

This topic tells you how to create a supply chain that performs a SonarQube scan. The SonarQube scan
component currently only supports scanning Maven projects.

## <a id="prerequisites"></a> Prepare

To prepare:

- [Install Tanzu Supply Chain packages](../../../supply-chain/platform-engineering/how-to/installing-supply-chain/install-authoring-profile.hbs.md#tsc-packages)
- [Install the Tanzu CLI](../../../install-tanzu-cli.hbs.md)
- [Install the Tanzu Supply Chain CLI plug-ins](../../../supply-chain/platform-engineering/how-to/install-the-cli.hbs.md)

## <a id="sonarqube-scan"></a> Create a supply chain with the SonarQube Scan component

To create a Supply Chain with the SonarQube Scan component by using the Tanzu Supply Chain CLI
plug-in:

1. Initialize Tanzu Supply Chain by running:

   ```console
   tanzu supplychain init --group GROUP-NAME
   ```

   Where `GROUP-NAME` is the group name of the supply chains. The default is
   `supplychains.tanzu.vmware.com`.

   For example:

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

1. As a platform engineer, use one of these options for generating a supply chain with SonarQube
   Scan:

   - [Generate SonarQube supply chain with overrides](#sonarqube-overrides): This option sets the
     SonarQube host URL and does not allow developers to provide a different host URL.

   - [Generate SonarQube supply chain with default](#sonarqube-defaults): This sets the default
     value for the SonarQube host URL, but developers can provide their own URL if needed.

### <a id="sonarqube-overrides"></a> Option 1: Generate SonarQube supply chain with overrides

To generate SonarQube supply chain with overrides:

1. Generate a supply chain by running:

   ```console
   tanzu supplychain generate --kind SUPPLY-CHAIN-NAME \
   --description DESCRIPTION \
   --component source-git-provider-1.0.0 \
   --component sonarqube-sast-scan-1.0.0 \
   --allow-overrides
   ```

   Where:

   - `SUPPLY-CHAIN-NAME` is the name of your supply chain.
   - `DESCRIPTION` is the description for your supply chain.

   For example:

   ```console
   $ tanzu supplychain generate --kind SonarQubeSC \
     --description SonarQube \
     --component source-git-provider-1.0.0 \
     --component sonarqube-sast-scan-1.0.0 \
     --allow-overrides

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

1. Open the generated YAML file `supplychains/sonarqubesc.yaml`, uncomment the overrides section
   with the `sonar-host-url`, and provide the host URL.

   Example supply chain YAML file:

    ```yaml
    apiVersion: supply-chain.apps.tanzu.vmware.com/v1alpha1
    kind: SupplyChain
    metadata:
      name: sonarqubesc
    spec:
      defines:
        group: example.com
        kind: SonarqubeSC
        plural: sonarqubescs
        version: v1alpha1
      description: ...
      stages:
        - componentRef:
            name: source-git-provider-1.0.0
          name: source-git-provider
        - componentRef:
            name: sonarqube-sast-scan-1.0.0
          name: sonarqube-sast-scan
      config:
        overrides:
          - path: spec.sonarqube.sonar-host-url
            value: "SONAR-URL"
      ...
    ```

   Where `SONAR-URL` is the URL for the SonarQube server to upload scan results to.

### <a id="sonarqube-defaults"></a> Option 2: Generate SonarQube supply chain with default values

To generate a SonarQube supply chain with default values:

1. Generate a supply chain by running:

   ```console
   tanzu supplychain generate --kind SUPPLY-CHAIN-NAME \
   --description DESCRIPTION \
   --component source-git-provider-1.0.0 \
   --component sonarqube-sast-scan-1.0.0 \
   --allow-defaults
   ```

   Where:

   - `SUPPLY-CHAIN-NAME` is the name of your supply chain.
   - `DESCRIPTION` is the description for your supply chain.

   For example:

   ```console
   $ tanzu supplychain generate --kind SonarQubeSC \
     --description SonarQube \
     --component source-git-provider-1.0.0 \
     --component sonarqube-sast-scan-1.0.0 \
     --allow-defaults

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

1. Open the generated YAML file `supplychains/sonarqubesc.yaml`, uncomment the `defaults` section
   with the `sonar-host-url`, and provide the host URL.

   Example supply chain YAML file:

    ```yaml
    apiVersion: supply-chain.apps.tanzu.vmware.com/v1alpha1
    kind: SupplyChain
    metadata:
      name: sonarqubesc
    spec:
      defines:
        group: example.com
        kind: SonarqubeSC
        plural: sonarqubescs
        version: v1alpha1
      description: ...
      stages:
        - componentRef:
            name: source-git-provider-1.0.0
          name: source-git-provider
        - componentRef:
            name: sonarqube-sast-scan-1.0.0
          name: sonarqube-sast-scan
      config:
        defaults:
          - path: spec.sonarqube.sonar-host-url
            value: "SONAR-URL"
      ...
    ```

   Where `SONAR-URL` is the URL for the SonarQube server to upload scan results to.

### <a id="apply-supply-chain"></a> Apply the supply chain

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

Where `DEV-NAMESPACE` is the namespace where the workload will run.

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
          image: maven:latest
          name: scan
    ...
    ```

1. Apply the custom task by running:

   ```console
   kubectl apply -f tasks/sonarqube-maven-scan.yaml -n DEV-NAMESPACE
   ```

   Where `DEV-NAMESPACE` is the namespace where the workload will be.