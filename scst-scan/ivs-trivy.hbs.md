# Configure an ImageVulnerabilityScan for Trivy

This topic gives you an example of how to configure an `ImageVulnerabilityScan` (IVS) for Trivy.

## <a id="example"></a> Example `ImageVulnerabilityScan`

This section gives you an example IVS that uses Trivy to scan a targeted image and push the results
to the specified registry location. For information about the IVS specification, see
[Configuration options](ivs-create-your-own.hbs.md#img-vuln-config-options).

```yaml
apiVersion: app-scanning.apps.tanzu.vmware.com/v1alpha1
kind: ImageVulnerabilityScan
metadata:
  name: trivy-ivs
  annotations:
    app-scanning.apps.tanzu.vmware.com/scanner-name: Trivy
spec:
  image: TARGET-IMAGE
  scanResults:
    location: registry/project/scan-results
  serviceAccountNames:
    publisher: publisher
    scanner: scanner
  steps:
  - name: trivy
    image: TRIVY-SCANNER-IMAGE
    command: ["trivy"]
    args:
    - image
    - $(params.image)
    - --exit-code=0
    - --no-progress
    - --scanners=vuln
    - --format=cyclonedx
    - --output=$(params.scan-results-path)/scan.cdx.json
```

Where:

- `TARGET-IMAGE` is the image to be scanned. The digest must be specified.
- `TRIVY-SCANNER-IMAGE` is the image containing the Trivy CLI. For example, `aquasec/trivy:0.42.1`.
  For information about publicly available Trivy images, see
  [DockerHub](https://hub.docker.com/r/aquasec/trivy/tags). For more information about using the
  Trivy CLI, see the [Trivy documentation](https://github.com/aquasecurity/trivy).

> **Note** Trivy versions later than 0.42.1 are not supported because they output CycloneDX 1.5,
> which is not supported for ingestion.

## <a id="trivy-db-requirement"></a> Trivy database size requirement

The recommended `storageSize` for Trivy scans is 4Gi because of the size of the Trivy database. If
the `storageSize` is not sufficient, you might encounter a `no space left on device` error when
initializing the database in the scan pod.

Update `app-scanning-values-file.yaml` for the `app-scanning.apps.tanzu.vmware.com` package to
change the default `storageSize`. For more information, see the
[installation documentation](install-app-scanning.hbs.md#install-scst-app-scanning).

```console
scans:
  workspace:
    storageSize: 4Gi
```

If you do not want to set a default `storageSize` by updating the `app-scanning-values-file.yaml`,
you must specify the `spec.workspace.size` when creating each standalone `ImageVulnerabilityScan` or
embedded `ImageVulnerabilityScan` in a
[ClusterImageTemplate](clusterimagetemplates.hbs.md#create-clusterimagetemplate).

> **Caution** As a publicly maintained image that is built outside of VMware build systems, the
> image might not meet the security standards VMware has established. Review the image before use to
> ensure that it meets your organization's security and compliance policies. For the publicly
> available Trivy scanner CLI image, the CLI commands and parameters used are accurate at the time
> of documentation.