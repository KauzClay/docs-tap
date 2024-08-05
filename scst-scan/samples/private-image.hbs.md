# Sample private image scan for Supply Chain Security Tools - Scan

This example describes how you can perform a scan against an image in a private registry for Supply
Chain Security Tools (SCST) - Scan.

{{> 'partials/scst-scan/scan-1-0-deprecation' }}

## <a id="define-resources"></a> Define the resources

Defining resources consists of setting up a target image pull secret and then creating a private
image scan.

### <a id="set-up-target-secret"></a> Set up target image pull secret

To set up the target image pull secret:

1. See if the target image secret already exists. Typically, the target image secret is created when
   Tanzu Application Platform is installed. If the target image secret exists, skip to
   [Create the private image scan](#create-private-image-scan) later in this topic. Otherwise,
   proceed to the next step.

1. Create a secret containing the credentials used to pull the target image that you want to scan by
   running:

   ```console
   kubectl create secret docker-registry TARGET-REGISTRY-CREDENTIALS-SECRET \
     --docker-server=YOUR-REGISTRY-SERVER \
     --docker-username=YOUR-NAME \
     --docker-password=YOUR-PASSWORD \
     --docker-email=YOUR-EMAIL-ADDRESS \
     -n DEV-NAMESPACE
   ```

   Where:

   - `TARGET-REGISTRY-CREDENTIALS-SECRET` is the name of the secret that is created.
   - `DEV-NAMESPACE` is the developer namespace where the scanner is installed.
   - `YOUR-REGISTRY-SERVER` is the registry server that you want to use.
   - `YOUR-NAME` is the name associated with the secret.
   - `YOUR-PASSWORD` is the password associated with the secret.
   - `YOUR-EMAIL-ADDRESS` is the email address associated with the secret.

   For more information about creating a secret, see the
   [Kubernetes documentation](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/#create-a-secret-by-providing-credentials-on-the-command-line).

1. Edit `tap-values.yaml` to include the name of the secret. For example:

    ```yaml
    grype:
      targetImagePullSecret: "TARGET-REGISTRY-CREDENTIALS-SECRET"
    ```

1. Upgrade Tanzu Application Platform with the modified `tap-values.yaml` file by running:

   ```console
   tanzu package installed update tap -p tap.tanzu.vmware.com -v ${TAP-VERSION}  \
   --values-file tap-values.yaml -n tap-install
   ```

   Where `TAP-VERSION` is the Tanzu Application Platform version.

### <a id="create-private-image-scan"></a> Create the private image scan

Create `sample-private-image-scan.yaml` with the following content:

```yaml
---
apiVersion: scanning.apps.tanzu.vmware.com/v1beta1
kind: ImageScan
metadata:
  name: sample-private-image-scan
spec:
  registry:
    image: IMAGE-URL
  scanTemplate: private-image-scan-template
```

Where `IMAGE-URL` is the URL of an image in a private registry.

## <a id="set-up-watch"></a> (Optional) Set up a watch

Before deploying the resources to a user-specified namespace, set up a `watch` in another terminal
to view the progression by running:

```console
watch kubectl get sourcescans,imagescans,pods,taskruns,scantemplates,scanpolicies -n DEV-NAMESPACE
```

Where `DEV-NAMESPACE` is the developer namespace where the scanner is installed.

For more information, see [Observing and Troubleshooting](../observing.hbs.md).

## <a id="deploy-resources"></a> Deploy the resources

Deploy the resources by running:

```console
kubectl apply -f sample-private-image-scan.yaml -n DEV-NAMESPACE
```

Where `DEV-NAMESPACE` is the developer namespace where the scanner is installed.

## <a id="view-scan-results"></a> View the scan results

To view the scan results:

1. When the scan finishes, run:

   ```console
   kubectl describe imagescan sample-private-image-scan -n DEV-NAMESPACE
   ```

   Where `DEV-NAMESPACE` is the developer namespace where the scanner is installed.

1. Verify that `Status.Conditions` includes `Reason: JobFinished` and
   `Message: The scan job finished`. For more information, see
   [Viewing and Understanding Scan Status Conditions](../results.hbs.md).

## <a id="clean-up"></a> Clean up

Clean up by running:

```console
kubectl delete -f sample-private-image-scan.yaml -n DEV-NAMESPACE
```

Where `DEV-NAMESPACE` is the developer namespace where the scanner is installed.

## <a id="view-vuln-reports"></a> View vulnerability reports

After finishing the scans, view the vulnerability results by using the
[Security Analysis plug-in](../../tap-gui/plugins/sa-tap-gui.hbs.md).
