# Tanzu Application Platform release notes

This topic contains release notes for Tanzu Application Platform v{{ vars.url_version }}.

## <a id='1-10-0'></a> v1.10.0

**Release Date**: 21 May 2024

### <a id='1-10-0-whats-new'></a> What's new in Tanzu Application Platform v1.10

This release includes the following platform-wide enhancements.

#### <a id='1-10-0-new-platform-features'></a> New platform-wide features

- Feature Description.

#### <a id='1-10-0-new-components'></a> New components

- [COMPONENT-NAME-AND-LINK-TO-DOCS](): Component description.

---

### <a id='1-10-0-new-features'></a> v1.10.0 New features by component and area

This release includes the following changes, listed by component and area.

#### <a id='1-10-0-aws-services'></a> v1.10.0 Features: AWS Services

- Sets the default CompositionUpdatePolicy on Compositions to `Manual`. Previously, the default was `Automatic`.
- Removes the default version on RDS services.
- Updates Provider from v1.2.0 to v1.4.0.

#### <a id='1-10-0-bitnami-services'></a> v1.10.0 Features: Bitnami Services

- Introduces the package value `claim_namespace`, which enables you to create services in the same
  namespace as the originating claim. This is now the default behavior. Previously, services were
  created in new namespaces. You can set this value globally or on a specific service.
  For more information, see [Package values of Bitnami Services](bitnami-services/reference/package-values.hbs.md).

- Updates Helm Charts to the latest versions:
    - MySQL to v10.1.0
    - PostgreSQL to v15.2.0
    - RabbitMQ to v13.0.0
    - Redis to v19.0.2
    - MongoDB to v15.1.1
    - Kafka to v28.0.1


#### <a id='1-10-0-crossplane'></a> v1.10.0 Features: Crossplane

- Updates [Universal Crossplane](https://github.com/upbound/universal-crossplane) to v1.15.2.up-1.

    This update increases the default CPU limit from 100m to 500m and memory limits from 512Mi to 1024Mi
    for the Crossplane controller. This might affect clusters which are close to or already over-subscribed.

- Updates [provider-helm](https://github.com/crossplane-contrib/provider-helm) to v0.17.0.

---

### <a id='1-10-0-breaking-changes'></a> v1.10.0 Breaking changes

This release includes the following changes, listed by component and area.

#### <a id='bitnami-services-bc'></a> v1.10.0 Breaking changes: Bitnami Services

- Bitnami Services are now created by default in the same namespace as the original claim rather
  than in a new dedicated namespace. You can configure this by using the `claim_namespace` package value.

    Existing instances that use the default behavior are unaffected unless the you have overridden
    `compositionUpdatePolicy` to `Automatic` or changed the `compositionRevisionRef`.
    In that case, the the Bitnami instances are recreated when the package is upgraded.
    To prevent instances being recreated, you can set the Bitnami package values to `shared_namespace=""`
    and `claim_namespace=False`, which was the previous default.

---

### <a id='1-10-0-security-fixes'></a> v1.10.0 Security fixes

This release has the following security fixes, listed by component and area.

#### <a id='1-10-0-COMPONENT-NAME-fixes'></a> v1.10.0 Security fixes: COMPONENT-NAME

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

### <a id='1-10-0-resolved-issues'></a> v1.10.0 Resolved issues

The following issues, listed by component and area, are resolved in this release.

#### <a id='1-10-0-services-toolkit-ri'></a> v1.10.0 Resolved issues: Services Toolkit

- Fixed an issue in which the Services Toolkit controller manager deleted Carvel SecretExport resources
  that it did not own.

#### <a id='1-10-0-ssc-ui-ri'></a> v1.10.0 Resolved issues: Supply Chain UI

- Performance issues in Supply Chain UI where the workloads page and workload details page would take long time to load are resolved

---

### <a id='1-10-0-known-issues'></a> v1.10.0 Known issues

This release has the following known issues, listed by component and area.

#### <a id='1-10-0-supply-chain-ki'></a> v1.10.0 Known issues: Supply Chain

- Components cannot have more than one resumption defined. When there are multiple resumptions,
  `WorkloadRuns` are not correctly created after resumptions trigger changes. The current workaround
  is to assess all triggers in a single resumption.

- Tanzu Supply Chain currently does not include support for Red Hat OpenShift. This means
  you cannot individually install components for Tanzu Supply Chain and Managed Resource Controller.
  You also cannot install the Authoring profile that includes those components as standard. Support
  for Red Hat OpenShift is planned for a later release.

- Tanzu Supply Chain currently does not include support for CA certificates in
  the Out of the Box components. However, you can edit the components to support CA certificates and
  use them to construct a new Supply Chain. Support for CA certificates as standard is planned for
  future versions of Tanzu Supply Chain.

#### <a id='1-10-0-ssc-ui-ki'></a> v1.10.0 Known issues: Supply Chain UI

- In workload details page, Config writer step takes >20 sec to load when 150+ workloads are visualized in the Supply Chain UI.

---

### <a id='1-10-0-components'></a> v1.10.0 Component versions

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

### <a id='scst-scan-deprecations'></a> Supply Chain Security Tools - Scan 1.0 deprecation

- SCST - Scan 1.0 is deprecated, but it remains the documented default option for online
  installation. SCST - Scan 2.0 will be the default in Tanzu Application Platform v1.11. SCST - Scan
  1.0 will be removed in a future Tanzu Application Platform version.
  For more information, see [SCST - Scan versions](scst-scan/overview.hbs.md#scst-scan-feat).

- Deprecation description including the release when the feature will be removed.

### <a id='svcs-toolkit-deprecations'></a> Services Toolkit deprecations

- The following APIs are deprecated and are marked for removal in Tanzu Application Platform v1.11:
  - clusterexampleusages.services.apps.tanzu.vmware.com/v1alpha1
  - clusterresources.services.apps.tanzu.vmware.com/v1alpha1


---
