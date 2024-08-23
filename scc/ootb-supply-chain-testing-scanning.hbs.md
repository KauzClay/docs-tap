# Out of the Box Supply Chain with Testing and Scanning for Supply Chain Choreographer

This topic provides an overview of Out of the Box Supply Chain with Testing and Scanning for Supply
Chain Choreographer.

This package contains Cartographer Supply Chains that tie together a series of Kubernetes resources
that drive a developer-provided workload from source code to a Kubernetes configuration ready to be
deployed to a cluster. It contains supply chains that pass the source code through testing and
vulnerability scanning, and also the container image.

This package includes all the capabilities of the Out of the Box Supply Chain With Testing, but adds
source and image scanning using Grype.

Workloads that use source code or prebuilt images perform the following:

- Building from source code:

  1. Watching a Git Repository or local directory for changes
  2. Running tests from a developer-provided Tekton pipeline
  3. Scanning the source code for known vulnerabilities using Grype
  4. Building a container image out of the source code with Buildpacks
  5. Scanning the image for known vulnerabilities
  6. Applying operator-defined conventions to the container definition
  7. Deploying the application to the same cluster
  8. Alternatively, outputting a Carvel Package containing the application to a Git repository.
     See [Carvel Package Supply Chains](carvel-package-supply-chain.hbs.md).

- Using a prebuilt application image:

  1. Scanning the image for known vulnerabilities
  1. Applying operator-defined conventions to the container definition
  1. Creating a deliverable object for deploying the application to a cluster
  1. Alternatively, outputting a Carvel package containing the application to a Git repository.
     See [Carvel Package Supply Chains](carvel-package-supply-chain.hbs.md).

## <a id="prerequisites"></a> Prerequisites

To use this supply chain, verify that:

- Out of the Box Templates is installed.
- Out of the Box Supply Chain With Testing **is NOT installed**.
- Out of the Box Supply Chain With Testing and Scanning **is installed**.
- The developer namespace is configured with the objects according to
  [Out of the Box Supply Chain With Testing guidance](ootb-supply-chain-testing.md#developer-namespace).
  This supply chain exists in addition to the Supply Chain with testing.
- (Optional) [Out of the Box Delivery Basic](ootb-delivery-basic.html) is installed if you want to
  deploy the application to the same cluster as the workload and supply chains.

Verify that you have the supply chains with scanning, not with testing, installed by running:

```console
tanzu apps cluster-supply-chain list
```

```console
NAME                      LABEL SELECTOR
source-test-scan-to-url   apps.tanzu.vmware.com/has-tests=true,apps.tanzu.vmware.com/workload-type=web
source-to-url             apps.tanzu.vmware.com/workload-type=web
```

If you see `source-test-to-url` in the list, the setup is wrong. You must not have the
`source-test-to-url` installed at the same time as `source-test-scan-to-url`.

## <a id="developer-namespace"></a> Developer namespace

This example builds on the previous Out of the Box Supply Chain examples, so only additions are
included here.

To ensure that you configured the namespace correctly, it is important that the namespace has the
objects that you configured in the other supply chain setups:

- **registries secrets**: Kubernetes secrets of type `kubernetes.io/dockerconfigjson` that contain
  credentials for pushing and pulling the container images built by the supply chain and the
  installation of Tanzu Application Platform.

- **service account**: The identity to be used for any interaction with the Kubernetes API made by
  the supply chain.

- **rolebinding**: Grant to the identity the necessary roles for creating the resources prescribed
  by the supply chain.

  For more information about the preceding objects, see
  [Out of the Box Supply Chain Basic](ootb-supply-chain-basic.hbs.md).

- **Tekton pipeline**: A pipeline runs whenever the supply chain hits the stage of testing the
  source code.

  For more information, see
  [Out of the Box Supply Chain Testing](ootb-supply-chain-testing.hbs.md#developer-namespace).

#### <a id="multiple-pl"></a> Allow multiple Tekton pipelines in a namespace

{{> 'partials/multiple-pipelines' }}

## <a id="developer-workload"></a> Developer workload

With the ScanPolicy and ScanTemplate objects, with the required names set, submitted to the same
namespace where the workload is submitted, you are ready to submit your workload.

Regardless of the workflow being targeted, such as local development or GitOps, the workload
configuration details are the same as in Out of the Box Supply Chain Basic, except that you mark the
workload as having tests enabled.

For example:

```console
tanzu apps workload create tanzu-java-web-app \
  --git-branch main \
  --git-repo https://github.com/vmware-tanzu/application-accelerator-samples \
  --sub-path tanzu-java-web-app \
  --label apps.tanzu.vmware.com/has-tests=true \
  --label app.kubernetes.io/part-of=tanzu-java-web-app \
  --type web
```

```console
Create workload:
      1 + |---
      2 + |apiVersion: carto.run/v1alpha1
      3 + |kind: Workload
      4 + |metadata:
      5 + |  labels:
      6 + |    apps.tanzu.vmware.com/workload-type: web
      7 + |    apps.tanzu.vmware.com/has-tests: "true"
      8 + |    app.kubernetes.io/part-of: tanzu-java-web-app
      9 + |  name: tanzu-java-web-app
     10 + |  namespace: default
     11 + |spec:
     12 + |  source:
     13 + |    git:
     14 + |      ref:
     15 + |        branch: main
     16 + |      url: https://github.com/vmware-tanzu/application-accelerator-samples
     17 + |    subPath: tanzu-java-web-app
```

## <a id="using-scan-v1"></a> Scan Images using supply chain security tools - scan v1

By default out of the box testing and scanning supply chain provides scanning with Aqua `Trivy` scanner
as part of [supply chain security tools - scan 2.0](../scst-scan/scan-2-0.hbs.md). For backwards compatibility, it is possible
to use [Scan 1.0](../scst-scan/scan-1-0.hbs.md).  To do so, add the following to your `tap-values.yaml`:

```yaml
  ootb_supply_chain_testing_scanning:
    image_scanner_template_name: image-scanner-template
```

In that case further customization of scan component is possible as:

```yaml
ootb_supply_chain_testing_scanning:
  image_scanner_template_name: image-scanner-template
  scanning:
    image:
      policy: SCAN-POLICY
      template: SCAN-TEMPLATE
```

Where `SCAN-POLICY` and `SCAN-TEMPLATE` are the names of the `ScanPolicy` and `ScanTemplate`.

- When SCST - Scan 1.0 is used back in Supply Chain, to set workload parameters related to `ScanPolicy` and `ScanTemplate` through cli, use the following commands:

  ```console
  tanzu apps workload apply WORKLOAD --param "scanning_image_policy=SCAN-POLICY" -n DEV-NAMESPACE
  tanzu apps workload apply WORKLOAD --param "scanning_image_template=SCAN-TEMPLATE" -n DEV-NAMESPACE
  ```

  Where:

  - `WORKLOAD` is the name of the workload.
  - `SCAN-POLICY` and `SCAN-TEMPLATE` are the names of the `ScanPolicy` and `ScanTemplate`.
  - `DEV-NAMESPACE` is the developer namespace.

  For more information, see
  [Create or update a workload](../cli-plugins/apps/tutorials/create-update-workload.hbs.md).
  [Enforce compliance policy by using Open Policy Agent](../scst-scan/policies.hbs.md)
  [About source and image scans](../scst-scan/explanation.hbs.md#about-source-and-image-scans)
