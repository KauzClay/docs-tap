# Tanzu Application Platform release notes

This topic contains release notes for Tanzu Application Platform v{{ vars.url_version }}.

<!-- The below partial is in the docs-tap/partials/support-notices directory -->

{{> 'partials/support-notices/short-term-support' }}

## <a id='1-11-0'></a> v1.11.0

**Release Date**: 02 July 2024

### <a id='1-11-0-whats-new'></a> What's new in Tanzu Application Platform v1.11

This release includes the following platform-wide enhancements.

#### <a id='1-11-0-new-platform-features'></a> New platform-wide features

- Feature Description.

#### <a id='1-11-0-new-components'></a> New components

- [COMPONENT-NAME-AND-LINK-TO-DOCS](): Component description.

---

### <a id='1-11-0-new-features'></a> v1.11.0 New features by component and area

This release includes the following changes, listed by component and area.

#### <a id='1-11-0-COMPONENT-NAME'></a> v1.11.0 Features: COMPONENT-NAME

- Feature description.

---

### <a id='1-11-0-breaking-changes'></a> v1.11.0 Breaking changes

This release includes the following changes, listed by component and area.

#### <a id='1-11-0-tap-bc'></a> v1.11.0 Breaking changes: Tanzu Application Platform

- Tanzu Application Platform releases have migrated from VMware Tanzu Network to the
  Broadcom Support Portal and Broadcom registry.
  Using VMware Tanzu Network to install or upgrade Tanzu Application Platform is no longer supported.

  Before you upgrade Tanzu Application Platform, you must relocate the Tanzu Application Platform
  images from the Broadcom registry `tanzu.packages.broadcom.com` to your own registry. For instructions,
  see [Relocate images to a registry](install-online/profile.html#relocate-images).

---

### <a id='1-11-0-security-fixes'></a> v1.11.0 Security fixes

This release has the following security fixes, listed by component and area.

#### <a id='1-11-0-COMPONENT-NAME-fixes'></a> v1.11.0 Security fixes: COMPONENT-NAME

- [CVE-2023-1234](https://nvd.nist.gov/vuln/detail/CVE-2023-1234): Security fix description.

OR add HTML table

<table>
<thead>
<tr>
<th>Package name</th>
<th>Vulnerabilities resolved</th>
</tr>
</thead>
<tbody>
<tr>
<td>PACKAGE.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-xxxx-xxxx-xxxx">GHSA-xxxx-xxxx-xxxx</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-12345">CVE-2023-12345</a></li>
</ul></details></td>
</tr>
</tbody>
</table>

---

### <a id='1-11-0-resolved-issues'></a> v1.11.0 Resolved issues

The following issues, listed by component and area, are resolved in this release.

#### <a id='1-11-0-COMPONENT-NAME-ri'></a> v1.11.0 Resolved issues: COMPONENT-NAME

- Resolved issue description.

---

### <a id='1-11-0-known-issues'></a> v1.11.0 Known issues

This release has the following known issues, listed by component and area.

#### <a id='1-11-0-COMPONENT-NAME-ki'></a> v1.11.0 Known issues: COMPONENT-NAME

- Known issue description with link to workaround.

---

### <a id='1-11-0-components'></a> v1.11.0 Component versions

The following table lists the Tanzu Application Platform package versions included with this release.
For open source component versions in this Tanzu Application Platform release, see
[Open source component versions](oss-component-versions.hbs.md).

| Component Name                                     | Version |
| -------------------------------------------------- | ------- |
| API Auto Registration                              |         |
| API portal                                         |         |
| Application Accelerator                            |         |
| Application Configuration Service                  |         |
| Application Live View APIServer                    |         |
| Application Live View back end                     |         |
| Application Live View connector                    |         |
| Application Live View conventions                  |         |
| Application Single Sign-On                         |         |
| Artifact Metadata Repository Observer              |         |
| AWS Services                                       |         |
| Bitnami Services                                   |         |
| Carbon Black Scanner for SCST - Scan (beta)        |         |
| Cartographer Conventions                           |         |
| cert-manager                                       |         |
| Cloud Native Runtimes                              |         |
| Contour                                            |         |
| Crossplane                                         |         |
| Default Roles                                      |         |
| Developer Conventions                              |         |
| Enterprise Config Server                           |         |
| External Secrets Operator                          |         |
| Flux CD Source Controller                          |         |
| Grype Scanner for SCST - Scan                      |         |
| Local Source Proxy                                 |         |
| Managed Resource Controller (beta)                 |         |
| Namespace Provisioner                              |         |
| Out of the Box Delivery - Basic                    |         |
| Out of the Box Supply Chain - Basic                |         |
| Out of the Box Supply Chain - Testing              |         |
| Out of the Box Supply Chain - Testing and Scanning |         |
| Out of the Box Templates                           |         |
| Service Bindings                                   |         |
| Service Registry                                   |         |
| Services Toolkit                                   |         |
| Snyk Scanner for SCST - Scan (beta)                |         |
| Source Controller                                  |         |
| Spring Boot conventions                            |         |
| Spring Cloud Gateway                               |         |
| Supply Chain Choreographer                         |         |
| Supply Chain Security Tools - Policy Controller    |         |
| Supply Chain Security Tools - Scan                 |         |
| Supply Chain Security Tools - Scan 2.0             |         |
| Supply Chain Security Tools - Store                |         |
| Tanzu Application Platform Telemetry               |         |
| Tanzu Build Service                                |         |
| Tanzu CLI                                          |         |
| Tanzu Developer Portal                             |         |
| Tanzu Developer Portal Configurator                |         |
| Tanzu Supply Chain (beta)                          |         |
| Tekton Pipelines                                   |         |

---

## <a id='deprecations'></a> Deprecations

The following features, listed by component, are deprecated.
Deprecated features remain on this list until they are retired from Tanzu Application Platform.

### <a id='COMPONENT-NAME-deprecations'></a> COMPONENT-NAME deprecations

- Deprecation description including the release when the feature will be removed.

---

## <a id="linux-kernel-cves"></a> Linux Kernel CVEs

Kernel level vulnerabilities are regularly identified and patched by Canonical.
Tanzu Application Platform releases with available images, which might contain known vulnerabilities.
When Canonical makes patched images available, Tanzu Application Platform incorporates these fixed
images into future releases.

The kernel runs on your container host VM, not the Tanzu Application Platform container image.
Even with a patched Tanzu Application Platform image, the vulnerability is not mitigated until you
deploy your containers on a host with a patched OS. An unpatched host OS might be exploitable if
the base image is deployed.
