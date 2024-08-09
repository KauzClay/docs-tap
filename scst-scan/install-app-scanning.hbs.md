# Install Supply Chain Security Tools - Scan 2.0 in a cluster

This topic tells you how to install Supply Chain Security Tools (SCST) - Scan 2.0 without using a
Tanzu Application Platform profile. By default, SCST - Scan 2.0 is installed in the Build and Full
profiles.

> **Note** Follow the steps in this topic only if you do not want to use a profile to install SCST -
> Scan 2.0. For more information about profiles, see
> [Components and installation profiles for Tanzu Application Platform](../about-package-profiles.hbs.md#installation-profiles-in-tanzu-application-platform-v-varsurl_version).

## <a id="scst-app-scanning-prereqs"></a> Prerequisites

SCST - Scan 2.0 requires the following prerequisites:

- Complete all prerequisites to install Tanzu Application Platform. For more information, see
  [Prerequisites](../prerequisites.hbs.md).
- Install the [Tekton component](../tekton/install-tekton.hbs.md).
- To store results, install AMR and AMR Observer. Downstream Tanzu Application Platform services,
  such as Tanzu Developer Portal and the Tanzu CLI, depend on scan results stored in SCST - Store to
  display correctly. For more information, see
  [Artifact Metadata Repository Observer for SCST - Store](../scst-store/amr/install-amr-observer.hbs.md).

> **Note** If you are installing SCST - Scan 2.0 in a cluster with restricted Kubernetes Pod
> Security Standards, you must update configurations for the Tekton Pipelines package. For more
> information, see
> [Scanning in a cluster with restricted Kubernetes Pod Security Standards](app-scanning-troubleshooting.hbs.md#scanning-restricted-pss).

## <a id="configure-app-scanning"></a> Configure properties

When you install SCST - Scan 2.0, you can configure the following optional properties:

| Key                        | Default                  | Type    | Description                                                                                                                           |
|----------------------------|--------------------------|---------|---------------------------------------------------------------------------------------------------------------------------------------|
| `caCertData`               | `""`                     | string  | The custom certificates trusted by the scan's connections.                                                                            |
| `docker.import`            | `true`                   | Boolean | Import `docker.pullSecret` from another namespace (requires `secretgen-controller`). Set to `false` if the secret is already present. |
| `docker.pullSecret`        | `registries-credentials` | string  | Name of a Docker pull secret in the deployment namespace to pull the scanner images.                                                  |
| `scans.maxConcurrentScans` | `10`                     | integer | Maximum number of scans running at one time. Set to `0` to have no limit.                                                             |
| `scans.priorityClassName`  | `""`                     | string  | Name of a predefined `PriorityClass` to apply to scan pods.                                                                           |
| `workspace.storageSize`    | `2Gi`                    | string  | Size of the `PersistentVolume` that the Tekton `pipelineruns` uses.                                                                   |
| `workspace.storageClass`   | `""`                     | string  | Name of the storage class to use while creating the `PersistentVolume` claims used by Tekton `pipelineruns`.                          |

> **Note** If the `StorageClass` you select does not have a node limit but uses the node storage,
> such as `hostpath`, the nodes must have large enough disks. For example, if a scan creates a 2Gi
> volume on a `hostpath` type storage class, `2Gi * number of AMR images` indicates how much storage
> this cluster needs overall. `2Gi * number of AMR images / number of nodes` indicates how much
> storage each node needs.

## <a id="install-scst-app-scanning"></a> Install

To install SCST - Scan 2.0:

1. List version information for the package by running:

   ```console
   tanzu package available list app-scanning.apps.tanzu.vmware.com --namespace tap-install
   ```

   For example:

   ```console
   $ tanzu package available list app-scanning.apps.tanzu.vmware.com --namespace tap-install
   - Retrieving package versions for app-scanning.apps.tanzu.vmware.com...
       NAME                                VERSION              RELEASED-AT
       app-scanning.apps.tanzu.vmware.com  0.1.0             2023-03-01 20:00:00 -0400 EDT
   ```

2. (Optional) Edit the default installation settings:

   1. Retrieve the configurable settings:

      ```console
      tanzu package available get app-scanning.apps.tanzu.vmware.com/VERSION --values-schema \
      --namespace tap-install
      ```

      Where `VERSION` is your package version number. For example, `0.1.0`.

      For example:

      ```console
      $ tanzu package available get app-scanning.apps.tanzu.vmware.com/0.1.0 --values-schema --namespace tap-install
      | Retrieving package details for app-scanning.apps.tanzu.vmware.com/0.1.0...

        KEY                     DEFAULT                 TYPE     DESCRIPTION
        docker.import           true                    boolean  Import `docker.pullSecret` from another namespace (requires
                                                                  secretgen-controller). Set to false if the secret will already be present.
        docker.pullSecret       registries-credentials  string   Name of a Docker pull secret in the deployment namespace to pull the scanner
                                                                  images.
        workspace.storageSize   2Gi                     string   Size of the Persistent Volume to be used by the tekton pipelineruns
        workspace.storageClass                          string   Name of the storage class to use while creating the Persistent Volume Claims
                                                                  used by tekton pipelineruns
        caCertData                                      string   The custom certificates to be trusted by the scan's connections
      ```

   1. To edit any of the default installation settings, create an `app-scanning-values-file.yaml`
      and append the key-value pairs to be modified to the file. For example:

      ```yaml
      scans:
        workspace:
          storageSize: 200Mi
      ```

3. Install the package by running:

   ```console
   tanzu package install app-scanning --package-name app-scanning.apps.tanzu.vmware.com \
       --version VERSION \
       --namespace tap-install \
       --values-file app-scanning-values-file.yaml
   ```

   Where:

   - The `--values-file` flag is not needed if you did not edit the default installation settings.
   - `VERSION` is your package version number. For example, `0.1.0`.

   For example:

   ```console
   $ tanzu package install app-scanning --package app-scanning.apps.tanzu.vmware.com \
       --version 0.1.0 \
       --namespace tap-install \
       --values-file app-scanning-values-file.yaml

       Installing package 'app-scanning.apps.tanzu.vmware.com'
       Getting package metadata for 'app-scanning.apps.tanzu.vmware.com'
       Creating service account 'app-scanning-default-sa'
       Creating cluster admin role 'app-scanning-default-cluster-role'
       Creating cluster role binding 'app-scanning-default-cluster-rolebinding'
       Creating package resource
       Waiting for 'PackageInstall' reconciliation for 'app-scanning'
       'PackageInstall' resource install status: Reconciling
       'PackageInstall' resource install status: ReconcileSucceeded
   ```

## <a id="config-sa-reg-creds"></a> Configure service accounts and registry credentials

This section contains instructions for running a standalone `ImageVulnerabilityScan` or using
multiple registries.

If your vulnerability scanner image and the image that you are scanning, by using a default scanner
or your own scanner, are in private registries that differ from the Tanzu Application Platform
bundles registry then you must edit your scanner service account to include registry credentials for
these registries.

> **Important** Skip the rest of this topic and proceed to
> [Enable SCST - Scan 2.0 for default Test and Scan supply chains](integrate-app-scanning.hbs.md) if
> either of these use cases is true:
>
> - You are running an `ImageVulnerabilityScan` in the context of a supply chain.
> - You used Namespace Provisioner to provision your developer namespace. For more information,
>   see the [Namespace Provisioner documentation](../namespace-provisioner/default-resources.hbs.md).

To configure service accounts and registry credentials, SCST - Scan 2.0 requires the following
access:

| Registry                                    | Permission | Service Account | Example                       |
|:--------------------------------------------|:-----------|:----------------|:------------------------------|
| Tanzu Application Platform bundles registry | Read       | scanner         | `tanzu.packages.broadcom.com` |
| Target image registry                       | Read       | scanner         | `your-registry.io`            |
| Vulnerability scanner image registry        | Read       | scanner         | `your-registry.io`            |
| Scan results location registry              | Write      | publisher       | `your-registry.io`            |

- The Tanzu Application Platform bundles registry contains the Tanzu Application Platform bundles.
  This is the registry from the
  [Relocate images to a registry](../install-online/profile.hbs.md#relocate-images)
  section.

- The target image registry contains the image to scan. This registry credential is required if you are
  scanning a private image. The image to scan is called the `target image` or `TARGET-IMAGE`.

- The vulnerability scanner image registry contains your vulnerability scanner image. This is only
  needed if you are bringing your own scanner and your vulnerability scanner image is located in a
  private registry that is different from the Tanzu Application Platform bundles registry.

- The scan results location registry is where scan results are published.

To configure service accounts and registry credentials:

1. Create a secret `scanning-tap-component-read-creds` with read access to the registry containing
   the Tanzu Application Platform bundles. This pulls the SCST - Scan 2.0 images. You can place your
   vulnerability scanner image in the registry where you relocated the Tanzu Application Platform
   bundles.

   ```console
   read -s TAP_REGISTRY_PASSWORD
   kubectl create secret docker-registry scanning-tap-component-read-creds \
     --docker-username=TAP-REGISTRY-USERNAME \
     --docker-password=$TAP_REGISTRY_PASSWORD \
     --docker-server=TAP-REGISTRY-URL \
     -n DEV-NAMESPACE
   ```

   Where `DEV-NAMESPACE` is the developer namespace where scanning occurs.

2. If you are scanning a private target image, create a secret `target-image-read-creds` with read
   access to the registry containing that target image.

   > **Important** If you followed the directions for
   > [Install Tanzu Application Platform](../install-intro.hbs.md), you can skip this step and use
   > the `targetImagePullSecret` secret with your service account as referenced in your
   > `tap-values.yaml` file. For more information, see
   > [Full profile](../install-online/profile.hbs.md#full-profile).

   Run:

   ```console
   read -s REGISTRY_PASSWORD
   kubectl create secret docker-registry target-image-read-creds \
     --docker-username=REGISTRY-USERNAME \
     --docker-password=$REGISTRY_PASSWORD \
     --docker-server=REGISTRY-URL \
     -n DEV-NAMESPACE
   ```

3. Create a secret `write-creds` with write access to the registry for the scanner to upload the
   scan results to by running:

   ```console
   read -s WRITE_PASSWORD
   kubectl create secret docker-registry write-creds \
     --docker-username=WRITE-USERNAME \
     --docker-password=$WRITE_PASSWORD \
     --docker-server=DESTINATION-REGISTRY-URL \
     -n DEV-NAMESPACE
   ```

4. (Optional) If you are bringing your own vulnerability scanner and your vulnerability scanner
   image is in a private registry that is different from the registry containing your Tanzu
   Application Platform bundles, you must create a secret `vulnerability-scanner-image-read-creds`
   with read access to the registry. Run:

   ```console
   read -s WRITE_PASSWORD
   kubectl create secret docker-registry vulnerability-scanner-image-read-creds \
     --docker-username=WRITE-USERNAME \
     --docker-password=$WRITE_PASSWORD \
     --docker-server=REGISTRY-URL \
     -n DEV-NAMESPACE
   ```

5. Create a `scanner-sa.yaml` file that contains the service account `scanner`, which enables SCST -
   Scan 2.0 to pull both the vulnerability scanner image and target image.

    ```yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: scanner
      namespace: DEV-NAMESPACE
    imagePullSecrets:
    - name: scanning-tap-component-read-creds
    - name: vulnerability-scanner-image-read-creds # optional
    secrets:
    - name: target-image-read-creds
    ```

    Where:

    - `imagePullSecrets.name` includes the name of the secret used to pull the scan component from
      the registry. If you are bringing your own vulnerability scanner and the vulnerability scanner
      image is in a separate private registry, you must also include the name of the secret with
      those registry credentials.

    - `secrets.name` is the name of the secret used to pull the target image to scan. This is
      required if the image you are scanning is private.

   You use `scanner-sa.yaml` to attach the one or more read secrets created earlier and pull the
   Tanzu Application Platform bundles and, optionally, your vulnerability scanner image under
   `imagePullSecrets`. The read secret created earlier for your target image is attached under
   `secrets`.

6. Apply the service account to your developer namespace by running:

   ```console
   kubectl apply -f scanner-sa.yaml
   ```

7. Create a `publisher-sa.yaml` file containing the service account `publisher`, which enables SCST
   - Scan 2.0 to push the scan results to a user-specified registry.

    ```yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: publisher
      namespace: DEV-NAMESPACE
    imagePullSecrets:
    - name: scanning-tap-component-read-creds
    secrets:
    - name: write-creds
    ```

   Where:

   - `imagePullSecrets.name` is the name of the secret used to pull the scan component image from
     the registry.
   - `secrets.name` is the name of the secret used to publish the scan results.

8. Apply the service account to your developer namespace by running:

   ```console
   kubectl apply -f publisher-sa.yaml
   ```

## <a id="registry-retention-policy"></a> (Optional) Set up your registry retention policy

Although Tanzu Application Platform ingests scan artifacts into the Metadata Store, and stores
information such as packages and parsed vulnerabilities, only a pointer to the original SBOM
location is stored. The original SBOM generated by the scan is not preserved within the Metadata
Store. VMware recommends that you keep these original artifacts in accordance with your
organization's archival requirements.

If the registry specified to push scan results to supports retention policies, you can configure the
registry to delete old scan results automatically if your archival requirements permit that. Scan
result artifacts accumulate over time and can quickly consume hard disk space.

For information about configuring Harbor tag retention rules, see the
[Harbor documentation](https://goharbor.io/docs/2.5.0/working-with-projects/working-with-images/create-tag-retention-rules/#configure-tag-retention-rules).

For example, you can configure Harbor to retain a specific number of the most recently pushed
artifacts or retain the artifacts pushed within a specific number of past days.

Retention policy setup differs between registry providers. See your specific registry's
documentation to learn the configuration options.
