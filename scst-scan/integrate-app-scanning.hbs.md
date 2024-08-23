# Use Grype scanner with SCST - Scan 2.0

The default configuration for Out of the Box Supply Chain with Testing and Scanning is
SCST - Scan 2.0 using Trivy scanner.
This topic tells you how to use Grype scanner with Supply Chain Security Tools (SCST) - Scan 2.0.

## <a id="overview"></a> Overview

SCST - Scan 2.0 includes two integrations for container image scanners:

| Container Image Scanner | Documentation                                | Cluster Image Template Name      | Description                                          |
|:------------------------|:---------------------------------------------|:---------------------------------|:-----------------------------------------------------|
| Aqua Trivy              | [Link](https://aquasecurity.github.io/trivy) | `image-vulnerability-scan-trivy` | Recommended scanner for SCST - Scan 2.0              |
| Anchore Grype           | [Link](https://github.com/anchore/grype)     | `image-vulnerability-scan-grype` | Alternative to Trivy that is used in SCST - Scan 1.0 |

VMware recommends using Aqua Trivy scanner with Tanzu Application Platform for container image
scanning. If you want to remain consistent with the default scanner in SCST - Scan 1.0, Anchore
Grype is included as an open-source alternative. Additionally, you can build an integration for
extra scanners. For more information, see
[Bring your own scanner with SCST - Scan 2.0](bring-your-own-scanner.hbs.md).

## <a id="use-grype"></a> Use Grype scanner

By default SCST - Scan 2.0 is enabled in out-of-the-box supply chain using Trivy scanner.
To use Grype as the scanner with supply chain:

1. Update your `tap-values.yaml` file to specify the Grype `ClusterImageTemplate` as follows:

    ```yaml
    ootb_supply_chain_testing_scanning:
      image_scanner_template_name: image-vulnerability-scan-grype
    ```

1. Update your Tanzu Application Platform installation by running:

   ```console
   tanzu package installed update tap -p tap.tanzu.vmware.com -v TAP-VERSION  --values-file \
   tap-values.yaml -n tap-install
   ```

   Where `TAP-VERSION` is the version of Tanzu Application Platform installed.

1. Verify scanning works as expected by creating a workload. For more information,
   see [Verify scanning with a Supply Chain integration](verify-app-scanning-supply-chain.hbs.md).
