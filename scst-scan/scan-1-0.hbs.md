# Overview of Supply Chain Security Tools - Scan 1.0

This topic gives you an overview of use cases, features, and Common Vulnerabilities and Exposures
(CVEs) for Supply Chain Security Tools (SCST) - Scan 1.0

{{> 'partials/scst-scan/scan-1-0-deprecation' }}

## <a id="overview"></a> Overview

With SCST - Scan, you can build and deploy secure trusted software that complies with your corporate
security requirements. SCST - Scan provides scanning and gatekeeping capabilities that application
and DevSecOps teams can incorporate early in their path to production because it is a known industry
best practice for reducing security risk and ensuring more efficient remediation.

## <a id="use-cases"></a> Language support

For information about the languages and frameworks that are supported by Tanzu Application Platform
components, see the
[Language and framework support table](../about-package-profiles.hbs.md#language-support).

## <a id="use-cases"></a> Use cases

The following use cases apply to SCST - Scan:

- Use your scanner as a plug-in to scan source code repositories and images for known CVEs before
  deploying to a cluster.
- Identify CVEs by continuously scanning each new code commit or each new image built.
- Analyze scan results against user-defined policies by using Open Policy Agent.
- Produce vulnerability scan results and post them to the SCST - Store from where they are queried.

## <a id="scst-scan-feat"></a> SCST - Scan features

The following SCST - Scan features enable these use cases:

- Kubernetes controllers to run scan `TaskRuns`.
- CRDs for Image and Source Scan.
- CRD for a scanner plug-in. An example is available by using Anchore's Syft and Grype.
- CRD for policy enforcement.
- Enhanced scanning coverage by analyzing the Cloud Native Buildpack SBoMs that Tanzu Build Service
  images provide.

## <a id="scst-scan-note"></a> Vulnerability Scanner limitations

Vulnerability scanning is an important practice in DevSecOps and the benefits of it are widely
recognized and accepted, but it has limitations. The following examples illustrate the limits that
are prevalent in most scanners today.

### <a id="missed-cves"></a> Missed CVEs

One limit of all vulnerability scanners is that no single tool can find all CVEs. Some causes for
missed CVEs include:

- The scanner does not detect the vulnerability because the CVE databases have not recorded it yet.
- The scanner verifies different CVE sources based on the detected package type and OS.
- The scanner might not fully support a particular programming language, packaging system or
  manifest format.
- The scanner might not implement binary analysis or fingerprinting.
- The detected component does not always include a canonical name and vendor, requiring the scanner
  to infer and attempt fuzzy matching.
- When vendors register impacted software with NVD, the provided information might not match the
  values in the release artifacts perfectly.

### <a id="false-positives"></a> False positives

Vulnerability scanners cannot always access the information to accurately identify whether a CVE
exists. This often leads to an influx of false positives where the tool mistakenly flags something
as a vulnerability. Unless you specialize in security or are very familiar with a vulnerable
component, assessing and determining false positives is a challenging and time-consuming activity.
Some causes for a false positive flag include:

- A component is misidentified because of similar names.
- A subcomponent is identified as the parent component.
- A component is correctly identified but the impacted function is not on a reachable code path.
- A component’s impacted function is on a reachable code path but is not a concern because of the
  specific environment or configuration.
- The version of a component might be incorrectly flagged as impacted.
- The detected component does not include a canonical name and vendor, requiring the scanner to
  infer and attempt fuzzy matching.

### Mitigation measures

Although vulnerability scanning is not a perfect solution, it is an essential part of the process
for keeping your organization secure. Scan more continuously and comprehensively to identify and
remediate zero-day vulnerabilities quicker by:

- Scanning earlier in the development cycle to ensure that you can address issues more efficiently
  and not delay a release. Tanzu Application Platform includes security practices, such as source
  and container image vulnerability scanning, earlier in the path to production for application
  teams.
- Scanning any base images in use. Tanzu Application Platform image scanning can recognize and scan
  the OS packages from a base image.
- Scanning running software in test, stage, and production environments at a regular cadence.
- Generating accurate provenance at any level so that scanners have a complete picture of the
  dependencies to scan. This is where a software bill of materials (SBoM) comes into play. To help
  you automate this process, VMware Tanzu Build Service, leveraging Cloud Native Buildpacks,
  generates an SBoM for buildpack-based projects. Because this SBoM is generated during the
  image-building stage it is more accurate and complete than one generated earlier or later in the
  release life cycle. This is because it can highlight dependencies introduced at the time of
  building that might introduce potential for compromise.
- Scan by using multiple scanners to maximize CVE coverage.
- Practice keeping your dependencies up-to-date.
- Reduce the overall surface area of attack by:

  - Using smaller dependencies
  - Reducing the number of third-party dependencies where possible
  - Using distroless base images where possible

- Maintain a central record of false positives to ease CVE triaging and remediation efforts.