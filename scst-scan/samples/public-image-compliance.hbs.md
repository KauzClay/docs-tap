# Sample public image scan with compliance check for Supply Chain Security Tools - Scan

This topic includes an example public image scan with a compliance check for Supply Chain Security
Tools (SCST) - Scan.

{{> 'partials/scst-scan/scan-1-0-deprecation' }}

## <a id="public-image-scan"></a> Public image scan

The following example performs an image scan on an image in a public registry. This image revision
has 223 known vulnerabilities (CVEs), spanning a number of severities. `ImageScan` uses the
`ScanPolicy` to run a compliance check against the CVEs.

The policy in this example is set to only consider `Critical` severity CVEs as a violation, which
returns 21 Critical Severity Vulnerabilities.

> **Caution** This example `ScanPolicy` is deliberately constructed to showcase the features
> available and must not be considered an acceptable base policy.

In this example, the scan:

- Finds all 223 of the CVEs
- Ignores any CVEs with severities that are not critical
- Indicates in `Status.Conditions` that 21 CVEs have violated policy compliance

### <a id="define-scanpolicy-imgscan"></a> Define the `ScanPolicy` and `ImageScan`

Create `sample-public-image-scan-with-compliance-check.yaml` with the following content:

```yaml
---
apiVersion: scanning.apps.tanzu.vmware.com/v1beta1
kind: ScanPolicy
metadata:
  name: sample-scan-policy
  labels:
    'app.kubernetes.io/part-of': 'enable-in-gui'
spec:
  regoFile: |
    package main

    # Accepted Values: "Critical", "High", "Medium", "Low", "Negligible", "UnknownSeverity"
    notAllowedSeverities := ["Critical"]
    ignoreCves := []

    contains(array, elem) = true {
      array[_] = elem
    } else = false { true }

    isSafe(match) {
      severities := { e | e := match.ratings.rating.severity } | { e | e := match.ratings.rating[_].severity }
      some i
      fails := contains(notAllowedSeverities, severities[i])
      not fails
    }

    isSafe(match) {
      ignore := contains(ignoreCves, match.id)
      ignore
    }

    deny[msg] {
      comps := { e | e := input.bom.components.component } | { e | e := input.bom.components.component[_] }
      some i
      comp := comps[i]
      vulns := { e | e := comp.vulnerabilities.vulnerability } | { e | e := comp.vulnerabilities.vulnerability[_] }
      some j
      vuln := vulns[j]
      ratings := { e | e := vuln.ratings.rating.severity } | { e | e := vuln.ratings.rating[_].severity }
      not isSafe(vuln)
      msg = sprintf("CVE %s %s %s", [comp.name, vuln.id, ratings])
    }

---
apiVersion: scanning.apps.tanzu.vmware.com/v1beta1
kind: ImageScan
metadata:
  name: sample-public-image-scan-with-compliance-check
spec:
  registry:
    image: "nginx:1.16"
  scanTemplate: public-image-scan-template
  scanPolicy: sample-scan-policy
```

### <a id="set-up-watch"></a> (Optional) Set up a watch

Before deploying the resources to a user-specified namespace, set up a `watch` in another terminal
to view the progression by running:

```console
watch kubectl get sourcescans,imagescans,pods,taskruns,scantemplates,scanpolicies -n DEV-NAMESPACE
```

Where `DEV-NAMESPACE` is the developer namespace where the scanner is installed.

For more information about setting up a watch, see
[Observing and Troubleshooting](../observing.hbs.md).

### <a id="deploy-resources"></a> Deploy the resources

Deploy the resources by running:

```console
kubectl apply -f sample-public-image-scan-with-compliance-check.yaml -n DEV-NAMESPACE
```

Where `DEV-NAMESPACE` is the developer namespace where the scanner is installed.

### <a id="view-scan-results"></a> View the scan results

To view the scan status:

1. After the scan has finished, run:

   ```console
   kubectl describe imagescan sample-public-image-scan-with-compliance-check -n DEV-NAMESPACE
   ```

   Where `DEV-NAMESPACE` is the developer namespace where the scanner is installed.

1. Verify that `Status.Conditions` includes `Reason: EvaluationFailed` and
   `Message: Policy violated because of 21 CVEs`.

   For more information about scan status conditions, see
   [Viewing and Understanding Scan Status Conditions](../results.hbs.md).

### <a id="modify-scanpolicy"></a> Edit `ScanPolicy`

To edit `ScanPolicy`, update the `ignoreCVEs` array in `ScanPolicy` to include the CVEs to ignore by
running:

   ```console
   ...
   spec:
     regoFile: |
       package policies

       default isCompliant = false

       # Accepted Values: "UnknownSeverity", "Critical", "High", "Medium", "Low", "Negligible"
       violatingSeverities := ["Critical"]
       # Adding the failing CVEs to the ignore array
       ignoreCVEs := ["CVE-2018-14643", "GHSA-f2jv-r9rf-7988", "GHSA-w457-6q6x-cgp9", "CVE-2021-23369", "CVE-2021-23383", "CVE-2020-15256", "CVE-2021-29940"]
   ...
   ```

### <a id="clean-up"></a> Clean up

Clean up by running:

```console
kubectl delete -f sample-public-image-scan-with-compliance-check.yaml -n DEV-NAMESPACE
```

Where `DEV-NAMESPACE` is the developer namespace where the scanner is installed.
