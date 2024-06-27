# Tanzu Application Platform release notes

This topic contains release notes for Tanzu Application Platform v{{ vars.url_version }}.

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

#### <a id='1-11-0-cnr'></a> v1.11.0 Features: Cloud Native Runtimes

- Upgrading Kubernetes requires draining nodes, which increases the risk of downtime. The following
  enhancements minimize the chances of traffic disruptions during upgrades:

  - Pod anti-affinity rules are added to the Activator deployment.

  - Pod anti-affinity rules are set by default for all Knative services. You can find the
    configuration in the `config-deployment` ConfigMap.

#### <a id='1-11-0-scc'></a> v1.11.0 Features: Supply Chain Choreographer

- [Carvel Package Supply Chains](scc/carvel-package-supply-chain.hbs.md) are now generally available
  (GA).

#### <a id='1-11-0-scst'></a> v1.11.0 Features: Supply Chain Security Tools

- [Vulnerability policy enforcement for Scan 2.0](scst-scan/creating-scan-policy-template.hbs.md):
  Policies that prevent the supply chain from proceeding if a vulnerability is detected are now
  defined.

#### <a id='1-11-0-tdp'></a> v1.11.0 Features: Tanzu Developer Portal

- **Supply Chain UI plug-in:**

  You can visualize your Carvel package deployments details and URLs for both server and
  web-type workloads.
  Ensure that your deliverable name is the same as the workload name for deliverables to be visualized
  correctly in Supply Chain UI.
  For more information, see [Visualize Carvel package deployment details](tap-gui/plugins/scc-tap-gui.hbs.md#visualize-carvel-dep).

---

### <a id='1-11-0-breaking-changes'></a> v1.11.0 Breaking changes

This release includes the following changes, listed by component and area.

#### <a id='1-11-0-tap-bc'></a> v1.11.0 Breaking changes: Tanzu Application Platform

- Tanzu Application Platform releases have migrated from VMware Tanzu Network to the
  Broadcom Support Portal and Broadcom registry.
  Using VMware Tanzu Network to install or upgrade Tanzu Application Platform is no longer supported.

  Before you upgrade, you must relocate the Tanzu Application Platform images from the Broadcom registry
  `tanzu.packages.broadcom.com` to your own registry.
  Make sure you relocate the images to your container image registry as part of the instructions in
  [Upgrade Tanzu Application Platform](upgrading.hbs.md).

#### <a id='1-11-0-cb-scanner-bc'></a> v1.11.0 Breaking changes: Carbon Black for Supply Chain Security Tools - Scan v1.0

- VMware Carbon Black for Supply Chain Security Tools - Scan v1.0 is now removed.

#### <a id='1-11-0-tanzu-cli-bc'></a> v1.11.0 Breaking changes: Tanzu CLI

- The Tanzu Insight plug-in is now removed.

#### <a id='1-11-0-tdp-bc'></a> v1.11.0 Breaking changes: Tanzu Developer Portal

- Tanzu Developer Portal Configurator is now removed.

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

### <a id='cnrs-deprecations'></a> Cloud Native Runtimes deprecations

- **`default_tls_secret` config option**: This config option is now in `contour.default_tls_secret`
  and is marked for removal in Cloud Native Runtimes. In the meantime, both options are
  supported, and `contour.default_tls_secret` takes precedence over `default_tls_secret`.

- **`ingress.[internal/external].namespace` config options**: These config options are now in
  `contour.[internal/external].namespace` are marked for removal in Cloud Native Runtimes. In
  the meantime, both options are supported, and `contour.[internal/external].namespace` takes
  precedence over `ingress.[internal/external].namespace`.

### <a id='svc-toolkit-deprecations'></a> Services Toolkit deprecations

- The following APIs are deprecated and are marked for removal in a future Tanzu Application Platform release:
  - `clusterexampleusages.services.apps.tanzu.vmware.com/v1alpha1`
  - `clusterresources.services.apps.tanzu.vmware.com/v1alpha1`

### <a id="sc-deprecations"></a> Source Controller deprecations

- The Source Controller `ImageRepository` API is deprecated and is marked for removal. Use the
  `OCIRepository` API instead. The Flux Source Controller installation includes the `OCIRepository`
  API. For more information about the `OCIRepository` API, see the
  [Flux documentation](https://fluxcd.io/flux/components/source/ocirepositories/).

### <a id='scst-policy-deprecations'></a> Supply Chain Security Tools - Policy Controller deprecation

- The [Policy Controller](scst-policy/overview.hbs.md) component is deprecated. VMware plans to
  remove it in a future Tanzu Application Platform version.

### <a id='scst-scan-deprecations'></a> Supply Chain Security Tools - Scan v1.0 deprecation

- SCST - Scan v1.0 is deprecated, but it remains the default option for online installation. SCST -
  Scan v2.0 will be the default in Tanzu Application Platform v1.11. SCST - Scan v1.0 will be
  removed in a future Tanzu Application Platform version. For more information, see
  [SCST - Scan versions](scst-scan/overview.hbs.md#scst-scan-feat).

### <a id='scst-store-deprecations'></a> Supply Chain Security Tools - Store deprecations

- The [metadata-store (MDS)](scst-store/mds-overview.hbs.md) component within SCST - Store is
  deprecated and is marked for removal in a future Tanzu Application Platform version.

### <a id="tekton-deprecations"></a> Tekton Pipelines deprecations

- Tekton `ClusterTask` is deprecated and marked for removal. Use the `Task` API instead. For more
  information, see the [Tekton documentation](https://tekton.dev/docs/pipelines/deprecations/).

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
