# Overview of Supply Chain Security Tools - Scan

This topic gives you an overview of use cases, features, and Common Vulnerabilities and Exposures
(CVEs) for Supply Chain Security Tools (SCST) - Scan.

## <a id="overview"></a> Overview

With Supply Chain Security Tools (SCST) - Scan, you can build and deploy secure trusted software
that complies with your corporate security requirements. SCST - Scan provides scanning and
gatekeeping capabilities that application and DevSecOps teams can incorporate early in their path to
production. This is the best practice for reducing security risk and ensuring more efficient
remediation.

## <a id="use-cases"></a> Language support

For information about the languages and frameworks that are supported by Tanzu Application Platform
components, see the
[Language and framework support table](../about-package-profiles.hbs.md#language-support).

## <a id="use-cases"></a> Use cases

Use cases for SCST - Scan include:

- Scan source code repositories and images for known CVEs as part of your software supply chain at
  build time.
- Scan container images produced by your supply chain and any running image in your Tanzu
  Application Platform clusters for newly reported vulnerabilities after the initial image build
  scan.
- Use one of the available scan integrations or create your own to use your existing vulnerability
  scanning platforms.
- Analyze supply chain scan results against user-defined policies.
- Store vulnerability scan results in SCST - Store for long-term archival and reporting.

## <a id="scst-scan-feat"></a> SCST - Scan versions

The two versions of SCST - Scan are SCST - Scan 1.0 and SCST - Scan 2.0.

### SCST - Scan 1.0

SCST - Scan 1.0 has been in the testing and scanning supply chain since it was introduced in Tanzu
Application Platform v1.0. SCST - Scan 1.0 can scan a workload for vulnerabilities, submit the scan
results to SCST - Store for long-term storage and reporting, and compare the results against a
policy defined by the user. This is all included in a tightly coupled scan job that is executed as
part of the testing and scanning supply chain.

This tight coupling of the capabilities made it difficult to develop and maintain integrations for
the vast ecosystem of vulnerability scanning platforms. To simplify this integration process, VMware
introduced SCST - Scan 2.0.

Although other scan integrations are available, the default configuration for SCST - Scan 1.0 is the
open-source [Anchore Grype](https://anchore.com/opensource/).

### SCST - Scan 2.0

SCST - Scan 2.0 was introduced in Tanzu Application Platform v1.5 while in alpha testing. SCST -
Scan 2.0 reached the general availability stage in Tanzu Application Platform v1.8. This iteration
of SCST - Scan focuses decoupling SCST - Store submission and policy from the scanning task. This
simplifies integration and gives more focus to the task of scanning workloads for vulnerabilities.

SCST - Scan 2.0 can scan container images after initially creating the workload. This allows
you to see the security posture of images as new vulnerabilities are reported. For
more information, see [Recurring Scanning](recurring-scanning.hbs.md). This capability can be used
with Scan 1.0 or Scan 2.0.

SCST - Scan 2.0 includes the default configuration to use open-source
[Aqua Security Trivy](https://www.aquasec.com/products/trivy/) as the image scanner, and a Grype
template is also included for backward compatibility. Examples of other integrations, and how to
build your own integration with this simplified interface, are provided in
[other topics](bring-your-own-scanner.hbs.md).

### Determine which version to use

Tanzu Application Platform v{{ vars.tap_version }} supports both SCST - Scan v1.0 and SCST - Scan
v2.0. In a future version, SCST - Scan v1.0 will be deprecated and replaced with SCST - Scan v2.0.
To help facilitate this transition, VMware is slowly making SCST - Scan 2.0 the default component.
The default SCST - Scan version varies depending on the environment Tanzu Application Platform is
installed on:

| Installation Environment                                | Default Component | Description                                                                               |
|---------------------------------------------------------|-------------------|-------------------------------------------------------------------------------------------|
| [Online Installation](../install-online/intro.hbs.md)   | SCST - Scan v1.0  | SCST - Scan v1.0 remains the default with the option to move to Scan 2.0.                 |
| [Offline Installation](../install-offline/intro.hbs.md) | SCST - Scan v2.0  | SCST - Scan v2.0 is the default because of a simplified air-gapped experience with Trivy. |
| [Azure Installation](../install-azure/intro.hbs.md)     | SCST - Scan v1.0  | SCST - Scan v1.0 remains the default with the option to move to Scan v2.0.                |
| [AWS Installation](../install-aws/intro.hbs.md)         | SCST - Scan v1.0  | SCST - Scan v1.0 remains the default with the option to move to SCST - Scan v2.0.         |

If you require a policy to block a workload in a supply-chain based on detected vulnerabilities, use
SCST - Scan v1.0.

If you want to create a scan integration for a scan tool that does not exist, use SCST - Scan v2.0
because it greatly simplifies the process. For more information, see
[Bring your own scanner to use with Supply Chain Security Tools - Scan 2.0](bring-your-own-scanner.hbs.md).

If you are using the Tanzu Supply Chain component, use SCST - Scan v2.0 because Tanzu Supply Chain
supports only SCST - Scan v2.0. For more information, see
[Overview of Tanzu Supply Chain](../supply-chain/about.hbs.md).

## <a id="scst-scan-note"></a> Vulnerability Scanner limitations

Vulnerability scanning is an important practice in DevSecOps and the benefits of it are widely
recognized and accepted, but it has limitations. The following examples illustrate the limits that
are prevalent in most scanners today.

### <a id="missed-cves"></a> Missed CVEs

One limit of all vulnerability scanners is that no single tool can find all CVEs. Some causes for
missed CVEs include:

- The scanner does not detect the vulnerability because the CVE databases have not recorded it yet.
- The scanner verifies different CVE sources based on the detected package type and OS.
- The scanner might not fully support a particular programming language, packaging system, or
  manifest format.
- The scanner might not implement binary analysis or fingerprinting.
- The detected component does not always include a canonical name and vendor, requiring the scanner
  to infer and attempt fuzzy matching.
- When vendors register impacted software with NVD, the provided information might not match the
  values in the release artifacts perfectly.

#### <a id="false-positives"></a> False positives

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

#### Mitigation measures

Although vulnerability scanning is not a perfect solution, it is an essential part of the process
for keeping your organization secure. To maximize the benefits while minimizing the impact of the
limits of vulnerability scanning, take the following steps to scan more continuously and
comprehensively to identify and remediate zero-day vulnerabilities quickly:

- Scan early in the development cycle to ensure that you can address issues more efficiently. Tanzu
  Application Platform includes security practices, such as source and container image vulnerability
  scanning, earlier in the path to production for application teams.
- Scan any base images in use. Tanzu Application Platform image scanning can recognize and scan the
  OS packages from a base image.
- Scan running software in test, stage, and production environments at a regular cadence.
- Generate accurate provenance at any level so that scanners have a complete picture of the
  dependencies to scan. This is where a software bill of materials (SBoM) is used. To help you
  automate this process, VMware Tanzu Build Service, leveraging Cloud Native Buildpacks, generates
  an SBoM for buildpack-based projects. Because this SBoM is generated during the image-building
  stage, it is more accurate and complete than one generated earlier or later in the release life
  cycle. This is because it can highlight dependencies introduced at the time of build that might
  introduce the potential for compromise.
- Use multiple scanners to maximize CVE coverage.
- Keep your dependencies up-to-date.
- Use smaller dependencies to reduce the overall surface area of attack.
- Reduce the number of third-party dependencies where possible.
- Use distroless base images when possible.
- Maintain a central record of false positives to ease CVE triaging and remediation efforts.
