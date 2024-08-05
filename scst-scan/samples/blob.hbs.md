# Sample public source scan of a blob for Supply Chain Security Tools - Scan

You can do a public source scan of a blob for Supply Chain Security Tools (SCST) - Scan. This
example performs a scan against source code in a `.tar.gz` file. This is helpful in a supply chain,
where there is a `GitRepository` step that handles cloning a repository and exporting the source
code as a compressed archive.

{{> 'partials/scst-scan/scan-1-0-deprecation' }}

## <a id="define-resources"></a> Define the resources

Create `public-blob-source-example.yaml` with this content:

```yaml
---
apiVersion: scanning.apps.tanzu.vmware.com/v1beta1
kind: SourceScan
metadata:
  name: public-blob-source-example
spec:
  blob:
    url: "https://gitlab.com/nina-data/ckan/-/archive/master/ckan-master.tar.gz"
  scanTemplate: blob-source-scan-template
```

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
kubectl apply -f public-blob-source-example.yaml -n DEV-NAMESPACE
```

Where `DEV-NAMESPACE` is the developer namespace where the scanner is installed.

## <a id="view-scan-results"></a> View the scan results

After the scan finishes, view the results:

1. Print the scan results by running:

```console
kubectl describe sourcescan public-blob-source-example -n DEV-NAMESPACE
```

Where `DEV-NAMESPACE` is the developer namespace where the scanner is installed.

1. Verify that `Status.Conditions` includes a `Reason: JobFinished` and
   `Message: The scan job finished`. For more information, see
   [Viewing and Understanding Scan Status Conditions](../results.hbs.md).

## <a id="clean-up"></a> Clean up

Clean up by running:

```console
kubectl delete -f public-blob-source-example.yaml -n DEV-NAMESPACE
```

Where `DEV-NAMESPACE` is the developer namespace where the scanner is installed.

## <a id="view-vuln-reports"></a> View vulnerability reports

After completing the scans, view the vulnerability results by using the
[Security Analysis plug-in](../../tap-gui/plugins/sa-tap-gui.hbs.md).
