# Tanzu Application Platform release notes

This topic contains release notes for Tanzu Application Platform v{{ vars.url_version }}.

## <a id='1-10-0'></a> v1.10.0

**Release Date**: 21 May 2024

### <a id='1-10-0-whats-new'></a> What's new in Tanzu Application Platform v1.10

This release includes the following platform-wide enhancements.

#### <a id='1-10-0-new-platform-features'></a> New platform-wide features

- Feature Description.

#### <a id='1-10-0-new-components'></a> New components

- [Enterprise Config Server](config-server/overview.hbs.md): Enterprise Config Server is an
  externalized configuration server based on the open-source Spring Cloud Config project. Config
  Server provides a centralized server for delivering external configuration properties to an
  application, and a central source for managing this configuration across deployment environments.

- [SonarQube Scan Supply Chain Component](scst-scan/tanzu-supply-chain/sonarqube-scan/sonarqube-supply-chain-overview.hbs.md):
  SonarQube Scan Supply Chain Component is a component that can be added to a supply chain to perform Static Application Security Testing(SAST) scans
  on the source code configured. This component is in alpha at this time, and only supports scanning for maven projects.

---

### <a id='1-10-0-new-features'></a> v1.10.0 New features by component and area

This release includes the following changes, listed by component and area.

#### <a id='1-10-0-app-accelerator'></a> v1.10.0 Features: Application Accelerator

- Provides an improved accelerator syntax and suthoring experience with a new Domain Specific Language (DSL) for authoring accelerators.
- Provides a Language Server in the local engine server that is used by the VS Code extension for syntax highlighting and code completion for accelerator authors using the new DSL.
- Adds a Spring Secure Resource Server sample accelerator, which provides a Multi-service demo application with Vue.js frontends backed by a secured Spring resource server.

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

#### <a id='1-10-0-cnr'></a> v1.10.0 Features: Cloud Native Runtimes

- Configuring CORS policy is now supported.

  <!-- See [Configure CORS policy for Cloud Native Runtimes](cloud-native-runtimes/how-to-guides/cors-policy.hbs.md) for details. This topic does not exist right now.-->

- L7 Routing to web workloads with TKGm and NSX ALB is now available. For more information, see
  [(Beta) Configure Tanzu Application Platform and VMware NSX Advanced Load Balancer to support L7 routing to web workloads](integrations/nsx-alb-tap-integration.hbs.md).

#### <a id='1-10-0-crossplane'></a> v1.10.0 Features: Crossplane

- Updates [Universal Crossplane](https://github.com/upbound/universal-crossplane) to v1.15.2.up-1.

    This update increases the default CPU limit from 100m to 500m and memory limits from 512Mi to 1024Mi
    for the Crossplane controller. This might affect clusters which are close to or already over-subscribed.

- Updates [provider-helm](https://github.com/crossplane-contrib/provider-helm) to v0.17.0.

#### <a id='1-10-0-java-buildpacks'></a> v1.10.0 Features: Java Buildpack supports CDS

- Introduces latest Spring Boot 3.3 optimization: CDS. Just adding an environment variable to the build section of your Spring Boot 3.3 workload, your app will start ~1.5x faster and will consume ~20% less memory. Lean everything about this optimization in the [Tanzu Java Buildpacks documentation](https://docs.vmware.com/en/VMware-Tanzu-Buildpacks/services/tanzu-buildpacks/GUID-java-java-buildpack.html#optimizations).

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

#### <a id='fluxcd-sc-bc'></a> v1.10.0 Breaking changes: FluxCD Source Controller

- FluxCD Source Controller updated the `GitRepository` API from `v1beta2` to `v1`.
  The controller accepts resources with API versions `v1beta1` and `v1beta2`, saving them as `v1`.

    **Unsupported fields for the `GitRepository` API:**

    - `spec.gitImplementation` is deprecated.
    `GitImplementation` defines the Git client library implementation.
    `go-git` is the default and only supported implementation. `libgit2`
    is no longer supported.
    - `spec.accessFrom` is deprecated. `AccessFrom`, which defines an Access
    Control List for enabling cross-namespace references to this object, was never
    implemented.
    - `status.contentConfigChecksum` is deprecated in favor of the explicit fields
    defined in the observed artifact content config within the status.
    - `status.artifact.checksum` is deprecated in favor of `status.artifact.digest`.
    - `status.url` is deprecated in favor of `status.artifact.url`.

    **Unsupported fields for the `OCIRepository` API:**

    - `status.contentConfigChecksum` is deprecated in favor of the explicit fields
      defined in the observed artifact content config within the status.

#### <a id='service-registry-bc'></a> v1.10.0 Breaking changes: Service Registry

- Mutual transport later security (mTLS) between Eureka peers and clients are now deactivated by
  default. You can activate mTLS by using the `tls` flag in the `EurekaServer` specification. Update
  existing instances with `tls: { activated: true }` to continue with mTLS activated.
  For more information, see
  [Create a EurekaServer resource](service-registry/configure-eureka-servers.hbs.md#create-eurekaserver).

----------------------------------------------------------------------------------------------------

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

#### <a id='1-10-0-supply-chain-ri'></a> v1.10.0 Resolved issues: Supply Chain

- Fixed performance issues in the Supply Chain UI that caused the Workloads page and Workload
  Details page to load slowly.

#### <a id='1-10-0-scst-scan-2-ri'></a> v1.10.0 Resolved issues: Supply Chain Security Tools - Scan 2.0

- Fixed an issue that overwrote the scanning-image value with a wrong default value for
  `ootb_supply_chain_testing_scanning.image_scanner_cli.image` when using Supply Chain Security
  Tools (SCST) - Scan 2.0 with a `ClusterImageTemplate` for templates other than the default Trivy
  template.

#### <a id='1-10-0-tdp-ri'></a> v1.10.0 Resolved issues: Tanzu Developer Portal

- **Runtime Resource View plug-in:**
  - Fixed the issue where the RRV plug-in was querying for resources across all namespaces despite
    the presence of a Backstage namespace annotation on the entity.
  - Fixed the issue where the RRV plug-in was displaying error messages for resources that did not
    exist on a cluster.

---

### <a id='1-10-0-known-issues'></a> v1.10.0 Known issues

This release has the following known issues, listed by component and area.

#### <a id='1-10-0-supply-chain-ki'></a> v1.10.0 Known issues: Application Accelerator

- When you create a Java project using the accelerator new project wizard in IntelliJ, it might not
  build correctly when first opened.

  This issue mostly occurs in Maven projects. When you open the new project for the first time,
  a dialog box might appear in the bottom right side of IntelliJ asking you to **Load Maven Project**.

  For a workaround, see [Troubleshoot Application Accelerator](./application-accelerator/troubleshooting.hbs.md#project-build-failure).

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

- In the Workload Details page, the config writer step takes longer than 20 seconds to load when
  more than 149 workloads are displayed in the Supply Chain UI.

#### <a id='1-10-0-scst-scan-2-ki'></a> v1.10.0 Known issues: Supply Chain Security Tools - Scan 2.0

- As of v1.10, when installing the Tanzu Application Platform Build profile or Full profile, Supply
  Chain Security Tools (SCST) - Scan 2.0 is also installed on the cluster. If you installed SCST -
  Scan 2.0 manually in an earlier Tanzu Application Platform version, uninstall SCST - Scan 2.0
  before upgrading to v1.10 to avoid conflict.

#### <a id='1-10-0-scst-store-ki'></a> v1.10.0 Known issues: Supply Chain Security Tools - Store

- When `observer.deploy_through_tmc` is `true`, properties are auto-configured for Tanzu Mission
  Control (TMC). This causes the `MultiClusterPropertyCollector` resource to overwrite existing
  Tanzu Application Platform values for Observer.

  When using Let's Encrypt ACME issuers, the resultant Kubernetes secret resource does
  not contain a `ca.crt` property. Therefore, when the `MultiClusterPropertyCollector` resource
  creates the Observer package configuration values secret, the required `ca_cert_data` is empty.

  To work around this issue, add the Certificate Authority (CA) Certificate to the
  `shared.ca_cert_data` key in the Tanzu Application Platform installation values.

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

### <a id='cnrs-deprecations'></a> Cloud Native Runtimes deprecations

- **`default_tls_secret` config option**: After changes in this release, this config option is moved
  to `contour.default_tls_secret`. `default_tls_secret` is marked for removal in Cloud Native Runtimes v2.7.
  In the meantime, both options are supported, and `contour.default_tls_secret` takes precedence over
  `default_tls_secret`.

- **`ingress.[internal/external].namespace` config options**: After changes in this release, these
  config options are moved to `contour.[internal/external].namespace`.
  `ingress.[internal/external].namespace` is marked for removal in Cloud Native Runtimes v2.7.
  In the meantime, both options are supported, and `contour.[internal/external].namespace` takes
  precedence over `ingress.[internal/external].namespace`.

### <a id='svc-toolkit-deprecations'></a> Services Toolkit deprecations

- The following APIs are deprecated and are marked for removal in Tanzu Application Platform v1.11:
  - `clusterexampleusages.services.apps.tanzu.vmware.com/v1alpha1`
  - `clusterresources.services.apps.tanzu.vmware.com/v1alpha1`

### <a id="sc-deprecations"></a> Source Controller deprecations

- The Source Controller `ImageRepository` API is deprecated and is marked for
  removal. Use the `OCIRepository` API instead.
  The Flux Source Controller installation includes the `OCIRepository` API.
  For more information about the `OCIRepository` API, see the
  [Flux documentation](https://fluxcd.io/flux/components/source/ocirepositories/).

### <a id='scst-scan-deprecations'></a> Supply Chain Security Tools - Scan v1.0 deprecation

- SCST - Scan v1.0 is deprecated, but it remains the default option for online installation. SCST -
  Scan v2.0 will be the default in Tanzu Application Platform v1.11. SCST - Scan v1.0 will be
  removed in a future Tanzu Application Platform version. For more information, see
  [SCST - Scan versions](scst-scan/overview.hbs.md#scst-scan-feat).

### <a id='scst-store-deprecations'></a> Supply Chain Security Tools - Store deprecations

- The [metadata-store (MDS)](scst-store/mds-overview.hbs.md) component within SCST - Store is
  deprecated as of Tanzu Application Platform v1.10.

### <a id='tanzu-cli-deprecations'></a> Tanzu CLI deprecations

- The [Tanzu Insight plug-in](cli-plugins/insight/cli-overview.hbs.md) for the Tanzu CLI is
  deprecated as of Tanzu Application Platform v1.10.

### <a id='tdp-deprecations'></a> Tanzu Developer Portal deprecations

- [Tanzu Developer Portal Configurator](tap-gui/configurator/about.hbs.md) is deprecated in Tanzu
  Application Platform v1.10 and will be removed in v1.11.

### <a id="tekton-deprecations"></a> Tekton Pipelines deprecations

- Tekton `ClusterTask` is deprecated and marked for removal. Use the `Task` API instead.
  For more information, see the [Tekton documentation](https://tekton.dev/docs/pipelines/deprecations/).
