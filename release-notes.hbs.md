# Tanzu Application Platform release notes

This topic contains release notes for Tanzu Application Platform v{{ vars.url_version }}.

## <a id='1-10-0'></a> v1.10.0

**Release Date**: 21 May 2024

### <a id='1-10-0-whats-new'></a> What's new in Tanzu Application Platform v1.10

This release includes the following platform-wide enhancements.

#### <a id='1-10-0-new-components'></a> New components

- [Enterprise Config Server](config-server/overview.hbs.md): Enterprise Config Server is an
  externalized configuration server based on the open-source Spring Cloud Config project. Config
  Server provides a centralized server for delivering external configuration properties to an
  application, and a central source for managing this configuration across deployment environments.

- [SonarQube Scan Tanzu Supply Chain component](scst-scan/tanzu-supply-chain/sonarqube-scan/overview.hbs.md):
  You can add the SonarQube Scan Tanzu Supply Chain component to a Tanzu Supply Chain to perform
  Static Application Security Testing (SAST) scans on the source code configured. This component is
  in the alpha stage, and only supports scanning for Maven projects.

---

### <a id='1-10-0-new-features'></a> v1.10.0 New features by component and area

This release includes the following changes, listed by component and area.

#### <a id='1-10-0-app-accelerator'></a> v1.10.0 Features: Application Accelerator

- Improves the accelerator syntax and authoring experience with a new Domain Specific Language (DSL)
  for authoring accelerators.

- Provides a Language Server in the local engine server that the VS Code extension uses for
  syntax highlighting and code completion for accelerator authors using the new DSL.

- Adds the Spring Secure Resource Server sample accelerator. This accelerator provides a multi-service
  demo application with Vue.js frontends and is backed by a secured Spring resource server.

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

- Supports configuring a cross-origin resource sharing (CORS) policy. For more information, see
  [Configure a CORS policy for Cloud Native Runtimes](cloud-native-runtimes/how-to-guides/cors-policy.hbs.md).

- (Beta) Supports L7 Routing to web workloads with TKGm and NSX ALB. For more information, see
  [(Beta) Configure Tanzu Application Platform and VMware NSX Advanced Load Balancer to support L7 routing to web workloads](integrations/nsx-alb-tap-integration.hbs.md).

#### <a id='1-10-0-crossplane'></a> v1.10.0 Features: Crossplane

- Updates [Universal Crossplane](https://github.com/upbound/universal-crossplane) to v1.15.2.up-1.

    This update increases the default CPU limit from 100m to 500m and memory limits from 512Mi to 1024Mi
    for the Crossplane controller. This might affect clusters which are close to or already over-subscribed.

- Updates [provider-helm](https://github.com/crossplane-contrib/provider-helm) to v0.17.0.

#### <a id='1-10-0-java-buildpacks'></a> v1.10.0 Features: Java Buildpack

- Introduces the latest Spring Boot v3.3 optimization: Class Data Sharing (CDS).
  Add an environment variable to the build section of your Spring Boot v3.3 workload to make
  your app to start approximately 1.5 times faster and consume approximately 20% less memory.
  For more information about this optimization, see the
  [Tanzu Java Buildpacks documentation](https://docs.vmware.com/en/VMware-Tanzu-Buildpacks/services/tanzu-buildpacks/GUID-java-java-buildpack.html#optimizations).

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

---

### <a id='1-10-0-security-fixes'></a> v1.10.0 Security fixes

This release has the following security fixes, listed by component and area.

<table>
<thead>
<tr>
<th>Package Name</th>
<th>Vulnerabilities Resolved</th>
</tr>
</thead>
<tbody>
<tr>
<td>accelerator.apps.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-4v7x-pqxf-cx7m">GHSA-4v7x-pqxf-cx7m</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-21012">CVE-2024-21012</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-39326">CVE-2023-39326</a></li>
</ul></details></td>
</tr>
<tr>
<td>amr-observer.apps.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-39326">CVE-2023-39326</a></li>
</ul></details></td>
</tr>
<tr>
<td>apiserver.appliveview.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24788">CVE-2024-24788</a></li>
</ul></details></td>
</tr>
<tr>
<td>aws.services.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-8r3f-844c-mc37">GHSA-8r3f-844c-mc37</a></li>
<li><a href="https://github.com/advisories/GHSA-4v7x-pqxf-cx7m">GHSA-4v7x-pqxf-cx7m</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2961">CVE-2024-2961</a></li>
</ul></details></td>
</tr>
<tr>
<td>cartographer.conventions.apps.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-4v7x-pqxf-cx7m">GHSA-4v7x-pqxf-cx7m</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-39326">CVE-2023-39326</a></li>
</ul></details></td>
</tr>
<tr>
<td>cartographer.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2961">CVE-2024-2961</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24785">CVE-2024-24785</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24784">CVE-2024-24784</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24783">CVE-2024-24783</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-0727">CVE-2024-0727</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-6237">CVE-2023-6237</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-6129">CVE-2023-6129</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-5678">CVE-2023-5678</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45290">CVE-2023-45290</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45289">CVE-2023-45289</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45288">CVE-2023-45288</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-39326">CVE-2023-39326</a></li>
</ul></details></td>
</tr>
<tr>
<td>cert-manager.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-c5q2-7r4c-mv6g">GHSA-c5q2-7r4c-mv6g</a></li>
<li><a href="https://github.com/advisories/GHSA-4v7x-pqxf-cx7m">GHSA-4v7x-pqxf-cx7m</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-39326">CVE-2023-39326</a></li>
</ul></details></td>
</tr>
<tr>
<td>cnrs.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-jq35-85cj-fj4p">GHSA-jq35-85cj-fj4p</a></li>
<li><a href="https://github.com/advisories/GHSA-8r3f-844c-mc37">GHSA-8r3f-844c-mc37</a></li>
<li><a href="https://github.com/advisories/GHSA-6wrf-mxfj-pf5p">GHSA-6wrf-mxfj-pf5p</a></li>
<li><a href="https://github.com/advisories/GHSA-4v7x-pqxf-cx7m">GHSA-4v7x-pqxf-cx7m</a></li>
<li><a href="https://github.com/advisories/GHSA-33pg-m6jh-5237">GHSA-33pg-m6jh-5237</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24785">CVE-2024-24785</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24784">CVE-2024-24784</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24783">CVE-2024-24783</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45290">CVE-2023-45290</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45289">CVE-2023-45289</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-39326">CVE-2023-39326</a></li>
</ul></details></td>
</tr>
<tr>
<td>connector.appliveview.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2961">CVE-2024-2961</a></li>
</ul></details></td>
</tr>
<tr>
<td>contour.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-8r3f-844c-mc37">GHSA-8r3f-844c-mc37</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-6780">CVE-2023-6780</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45284">CVE-2023-45284</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-39326">CVE-2023-39326</a></li>
</ul></details></td>
</tr>
<tr>
<td>controller.source.apps.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-4v7x-pqxf-cx7m">GHSA-4v7x-pqxf-cx7m</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-39326">CVE-2023-39326</a></li>
</ul></details></td>
</tr>
<tr>
<td>crossplane.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-qppj-fm5r-hxr3">GHSA-qppj-fm5r-hxr3</a></li>
<li><a href="https://github.com/advisories/GHSA-pwcw-6f5g-gxf8">GHSA-pwcw-6f5g-gxf8</a></li>
<li><a href="https://github.com/advisories/GHSA-mq39-4gv4-mvpx">GHSA-mq39-4gv4-mvpx</a></li>
<li><a href="https://github.com/advisories/GHSA-jq35-85cj-fj4p">GHSA-jq35-85cj-fj4p</a></li>
<li><a href="https://github.com/advisories/GHSA-hmfx-3pcx-653p">GHSA-hmfx-3pcx-653p</a></li>
<li><a href="https://github.com/advisories/GHSA-6xv5-86q9-7xr8">GHSA-6xv5-86q9-7xr8</a></li>
<li><a href="https://github.com/advisories/GHSA-6rx9-889q-vv2r">GHSA-6rx9-889q-vv2r</a></li>
<li><a href="https://github.com/advisories/GHSA-67fx-wx78-jx33">GHSA-67fx-wx78-jx33</a></li>
<li><a href="https://github.com/advisories/GHSA-53c4-hhmh-vw5q">GHSA-53c4-hhmh-vw5q</a></li>
<li><a href="https://github.com/advisories/GHSA-2wrh-6pvc-2jm9">GHSA-2wrh-6pvc-2jm9</a></li>
<li><a href="https://github.com/advisories/GHSA-2qjp-425j-52j9">GHSA-2qjp-425j-52j9</a></li>
<li><a href="https://github.com/advisories/GHSA-259w-8hf6-59c2">GHSA-259w-8hf6-59c2</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2961">CVE-2024-2961</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-29018">CVE-2024-29018</a></li>
</ul></details></td>
</tr>
<tr>
<td>fluxcd-source-controller.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-xw73-rw38-6vjc">GHSA-xw73-rw38-6vjc</a></li>
<li><a href="https://github.com/advisories/GHSA-vfp6-jrw2-99g9">GHSA-vfp6-jrw2-99g9</a></li>
<li><a href="https://github.com/advisories/GHSA-v554-xwgw-hc3w">GHSA-v554-xwgw-hc3w</a></li>
<li><a href="https://github.com/advisories/GHSA-v53g-5gjp-272r">GHSA-v53g-5gjp-272r</a></li>
<li><a href="https://github.com/advisories/GHSA-qppj-fm5r-hxr3">GHSA-qppj-fm5r-hxr3</a></li>
<li><a href="https://github.com/advisories/GHSA-mq39-4gv4-mvpx">GHSA-mq39-4gv4-mvpx</a></li>
<li><a href="https://github.com/advisories/GHSA-jw44-4f3j-q396">GHSA-jw44-4f3j-q396</a></li>
<li><a href="https://github.com/advisories/GHSA-jq35-85cj-fj4p">GHSA-jq35-85cj-fj4p</a></li>
<li><a href="https://github.com/advisories/GHSA-f4p5-x4vc-mh4v">GHSA-f4p5-x4vc-mh4v</a></li>
<li><a href="https://github.com/advisories/GHSA-c5q2-7r4c-mv6g">GHSA-c5q2-7r4c-mv6g</a></li>
<li><a href="https://github.com/advisories/GHSA-95pr-fxf5-86gv">GHSA-95pr-fxf5-86gv</a></li>
<li><a href="https://github.com/advisories/GHSA-8r3f-844c-mc37">GHSA-8r3f-844c-mc37</a></li>
<li><a href="https://github.com/advisories/GHSA-88jx-383q-w4qc">GHSA-88jx-383q-w4qc</a></li>
<li><a href="https://github.com/advisories/GHSA-7ww5-4wqc-m92c">GHSA-7ww5-4wqc-m92c</a></li>
<li><a href="https://github.com/advisories/GHSA-4v7x-pqxf-cx7m">GHSA-4v7x-pqxf-cx7m</a></li>
<li><a href="https://github.com/advisories/GHSA-45x7-px36-x8w8">GHSA-45x7-px36-x8w8</a></li>
<li><a href="https://github.com/advisories/GHSA-2c7c-3mj9-8fqh">GHSA-2c7c-3mj9-8fqh</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2961">CVE-2024-2961</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-29018">CVE-2024-29018</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24787">CVE-2024-24787</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24785">CVE-2024-24785</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24784">CVE-2024-24784</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24783">CVE-2024-24783</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45290">CVE-2023-45290</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45289">CVE-2023-45289</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45288">CVE-2023-45288</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-39326">CVE-2023-39326</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-39319">CVE-2023-39319</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-39318">CVE-2023-39318</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-29409">CVE-2023-29409</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-29406">CVE-2023-29406</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-24532">CVE-2023-24532</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2022-39272">CVE-2022-39272</a></li>
</ul></details></td>
</tr>
<tr>
<td>java-lite.buildpacks.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-mq39-4gv4-mvpx">GHSA-mq39-4gv4-mvpx</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-29018">CVE-2024-29018</a></li>
</ul></details></td>
</tr>
<tr>
<td>metadata-store.apps.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28835">CVE-2024-28835</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28834">CVE-2024-28834</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28085">CVE-2024-28085</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24785">CVE-2024-24785</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24784">CVE-2024-24784</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24783">CVE-2024-24783</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45290">CVE-2023-45290</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45289">CVE-2023-45289</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-39326">CVE-2023-39326</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2022-3715">CVE-2022-3715</a></li>
</ul></details></td>
</tr>
<tr>
<td>nodejs-lite.buildpacks.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-x84c-p2g9-rqv9">GHSA-x84c-p2g9-rqv9</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-32473">CVE-2024-32473</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24785">CVE-2024-24785</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24784">CVE-2024-24784</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24783">CVE-2024-24783</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45290">CVE-2023-45290</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45289">CVE-2023-45289</a></li>
</ul></details></td>
</tr>
<tr>
<td>ruby-lite.buildpacks.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-x84c-p2g9-rqv9">GHSA-x84c-p2g9-rqv9</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-32473">CVE-2024-32473</a></li>
</ul></details></td>
</tr>
<tr>
<td>service-registry.spring.apps.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-8r3f-844c-mc37">GHSA-8r3f-844c-mc37</a></li>
<li><a href="https://github.com/advisories/GHSA-2wrp-6fg6-hmc5">GHSA-2wrp-6fg6-hmc5</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24785">CVE-2024-24785</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24784">CVE-2024-24784</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24783">CVE-2024-24783</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-20945">CVE-2024-20945</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-20921">CVE-2024-20921</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-20919">CVE-2024-20919</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45290">CVE-2023-45290</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45289">CVE-2023-45289</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-39326">CVE-2023-39326</a></li>
</ul></details></td>
</tr>
<tr>
<td>services-toolkit.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-8r3f-844c-mc37">GHSA-8r3f-844c-mc37</a></li>
<li><a href="https://github.com/advisories/GHSA-45x7-px36-x8w8">GHSA-45x7-px36-x8w8</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-39326">CVE-2023-39326</a></li>
</ul></details></td>
</tr>
<tr>
<td>spring-cloud-gateway.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-xh6m-7cr7-xx66">GHSA-xh6m-7cr7-xx66</a></li>
<li><a href="https://github.com/advisories/GHSA-wjxj-5m7g-mg7q">GHSA-wjxj-5m7g-mg7q</a></li>
<li><a href="https://github.com/advisories/GHSA-gvpg-vgmx-xg6w">GHSA-gvpg-vgmx-xg6w</a></li>
<li><a href="https://github.com/advisories/GHSA-c5vj-wp4v-mmvx">GHSA-c5vj-wp4v-mmvx</a></li>
<li><a href="https://github.com/advisories/GHSA-8h4x-xvjp-vf99">GHSA-8h4x-xvjp-vf99</a></li>
<li><a href="https://github.com/advisories/GHSA-5gj6-62g7-vmgf">GHSA-5gj6-62g7-vmgf</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-33264">CVE-2023-33264</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2021-25740">CVE-2021-25740</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2020-8554">CVE-2020-8554</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2015-7561">CVE-2015-7561</a></li>
</ul></details></td>
</tr>
<tr>
<td>sso.apps.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-xfrj-6vvc-3xm2">GHSA-xfrj-6vvc-3xm2</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-39326">CVE-2023-39326</a></li>
</ul></details></td>
</tr>
<tr>
<td>tap-gui.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-32487">CVE-2024-32487</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2961">CVE-2024-2961</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28182">CVE-2024-28182</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26581">CVE-2024-26581</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52603">CVE-2023-52603</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52600">CVE-2023-52600</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-24023">CVE-2023-24023</a></li>
</ul></details></td>
</tr>
<tr>
<td>trivy.app-scanning.component.apps.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28835">CVE-2024-28835</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28834">CVE-2024-28834</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28085">CVE-2024-28085</a></li>
</ul></details></td>
</tr>
<tr>
<td>web-servers-lite.buildpacks.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-x84c-p2g9-rqv9">GHSA-x84c-p2g9-rqv9</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-32473">CVE-2024-32473</a></li>
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

#### <a id='1-10-0-scst-scan-2-ri'></a> v1.10.0 Resolved issues: Supply Chain Security Tools - Scan 2.0

- Fixed an issue that overwrote the scanning-image value with a wrong default value for
  `ootb_supply_chain_testing_scanning.image_scanner_cli.image` when using Supply Chain Security
  Tools (SCST) - Scan 2.0 with a `ClusterImageTemplate` for templates other than the default Trivy
  template.

#### <a id='1-10-0-tdp-ri'></a> v1.10.0 Resolved issues: Tanzu Developer Portal

- **Supply Chain UI plug-in:**

  - Fixed performance issues in the Supply Chain UI that caused the Workloads page and Workload Details
    page to load slowly.

- **Runtime Resource View plug-in:**

  - Fixed the issue where the RRV plug-in was querying for resources across all namespaces despite
    the presence of a Backstage namespace annotation on the entity.
  - Fixed the issue where the RRV plug-in was displaying error messages for resources that did not
    exist on a cluster.

---

### <a id='1-10-0-known-issues'></a> v1.10.0 Known issues

This release has the following known issues, listed by component and area.

#### <a id='1-10-0-tap-ki'></a> v1.10.0 Known issues: Tanzu Application Platform

- On Azure Kubernetes Service (AKS), the Datadog Cluster Agent cannot reconcile the webhook, which
  leads to an error.
  For troubleshooting information, see [Datadog agent cannot reconcile webhook on AKS](./troubleshooting-tap/troubleshoot-using-tap.hbs.md#datadog-agent-aks).

- The Tanzu Application Platform integration with Tanzu Service Mesh does not work
  on vSphere with TKR v1.26. For more information about this integration, see
  [Set up Tanzu Service Mesh](integrations/tsm-tap-integration.hbs.md).
  As a workaround, you can apply the label to update pod security on a TKr v1.26 Kubernetes namespace
  as advised by the release notes for
  [TKr 1.26.5 for vSphere 8.x](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-releases/services/rn/vmware-tanzu-kubernetes-releases-release-notes/index.html#TKr%201.26.5%20for%20vSphere%208.x-What's%20New).
  However, applying this label provides more than the minimum necessary privilege to the resources in
  developer namespaces.

#### <a id='1-10-0-api-auto-reg-ki'></a> v1.10.0 Known issues: API Auto Registration

- Registering conflicting `groupId` and `version` with API portal:

  If you create two `CuratedAPIDescriptor`s with the same `groupId` and `version` combination, both
  reconcile without throwing an error, and the `/openapi?groupId&version` endpoint returns both specifications.
  If you are adding both specifications to the API portal, only one of them might show up in the
  API portal UI with a warning indicating that there is a conflict.
  If you add the route provider annotation for both of the `CuratedAPIDescriptor`s to use Spring Cloud Gateway,
  the generated API specspecification includes API routes from both `CuratedAPIDescriptor`s.

  You can see the `groupId` and `version` information from all `CuratedAPIDescriptor`s by running:

    ```console
    $ kubectl get curatedapidescriptors -A

    NAMESPACE           NAME         GROUPID            VERSION   STATUS   CURATED API SPEC URL
    my-apps             petstore     test-api-group     1.2.3     Ready    http://AAR-CONTROLLER-FQDN/openapi/my-apps/petstore
    default             mystery      test-api-group     1.2.3     Ready    http://AAR-CONTROLLER-FQDN/openapi/default/mystery
    ```

- When creating an `APIDescriptor` with different `apiSpec.url` and `server.url`, the controller
  incorrectly uses the API spec URL as the server URL. To avoid this issue, use `server.url` only.

#### <a id='1-10-0-alm-ki'></a> v1.10.0 Known issues: App Last Mile Catalog

- The app-config-web, app-config-server, and app-config-worker components do not allow developers to
  override the default application ports.
  This means that applications that use non-standard ports do not work. To work around this, you can
  configure ports by providing values to the resulting Carvel package.
  This issue is planned to be fixed in a future release.

#### <a id='1-10-0-supply-chain-ki'></a> v1.10.0 Known issues: Application Accelerator

- When you create a Java project using the accelerator new project wizard in IntelliJ, it might not
  build correctly when first opened.

  This issue mostly occurs in Maven projects. When you open the new project for the first time,
  a dialog box might appear in the bottom right side of IntelliJ asking you to **Load Maven Project**.

  For a workaround, see [Troubleshoot Application Accelerator](./application-accelerator/troubleshooting.hbs.md#project-build-failure).

#### <a id='1-10-0-alv-ki'></a> v1.10.0 Known issues: Application Live View

- On the Run profile, Application Live View fails to reconcile if you use a non-default cluster issuer
while installing through Tanzu Mission Control.

#### <a id='1-10-0-amr-obs-ce-hndlr-ki'></a> v1.10.0 Known issues: Artifact Metadata Repository Observer and CloudEvent Handler

- Periodic reconciliation or restarting of the AMR Observer causes reattempted posting of
  ImageVulnerabilityScan results. There is an error on duplicate submission of identical
  ImageVulnerabilityScans you can ignore if the previous submission was successful.

#### <a id='1-10-0-bitnami-services-ki'></a> v1.10.0 Known issues: Bitnami Services

- If you try to configure private registry integration for the Bitnami Services
  after having already created a claim for one or more of the services using the default
  configuration, the updated private registry configuration does not appear to take effect.
  This is due to caching behavior in the system which is not accounted for during configuration updates.
  For a workaround, see [Troubleshoot Bitnami Services](bitnami-services/how-to-guides/troubleshooting.hbs.md#private-reg).


#### <a id='1-10-0-config-server-ki'></a> v1.10.0 Known issues: cert-manager

- On TKGs TGKr 1.27, cert-manager is unable to configure Gateway API support due to
the use of "gateway.networking.k8s.io/v1beta1".  As a workaround, either install the 
"gateway.networking.k8s.io/v1." APIs or disable the use of Gateway API in cert-manager 
by adding these lines to tap-values: 

```console
   cert_manager: 
    controller: 
      feature_gates: 
        ExperimentalGatewayAPISupport: false
```


#### <a id='1-10-0-carto-conventions'></a>v1.10.0 Known issues: Cartographer Conventions

- Before Tanzu Application Platform v1.9, the `cartographer.tanzu.vmware.com` package contained two products:
  Cartographer and Cartographer Conventions.
  In Tanzu Application Platform v1.9.0 the Cartographer Conventions product was removed from the
  `cartographer.tanzu.vmware.com` package and is distributed in its own package named
  `cartographer.conventions.apps.tanzu.vmware.com`.

  When you upgrade to Tanzu Application Platform v1.9 or later, an issue might occur when installing the new
  package for Cartographer Conventions. The upgrade might fail to reconcile and show error messages
  similar to the following:

    ```console
    Resource 'clusterrole/cartographer-conventions-manager-role (rbac.authorization.k8s.io/v1) cluster' is already associated with a different app 'cartographer.app'
    ```

  This message might appear more than once, and it can refer to several resources.

  These errors appear when kapp-controller on the cluster tries to install the new Cartographer
  Conventions package before the Cartographer package has reconciled.
  The new package for Cartographer Conventions tries to install resources that the existing
  Cartographer package still owns.

  Although it looks like the upgrade fails, if you wait a few minutes, kapp-controller finishes
  the installation and the packages will reconcile successfully. The system works normally after
  the reconciliation is complete.

  This error does not occur on a new installation of Tanzu Application Platform.

  <!-- remove in 1.11 - only affects n-2 users in 1.10 -->

- While processing workloads with large SBOMs, the Cartographer Convention controller manager pod can
  fail with the status `CrashLoopBackOff` or `OOMKilled`.
  For information about how to increase the memory limit for both the convention server and webhook
  servers, including app-live-view-conventions, spring-boot-webhook, and developer-conventions/webhook,
  see [Troubleshoot Cartographer Conventions](cartographer-conventions/troubleshooting.hbs.md).

#### <a id='1-10-0-crossplane-ki'></a> v1.10.0 Known issues: Crossplane

- The Crossplane `validatingwebhookconfiguration` is not removed when you uninstall the
  Crossplane package.
  To workaround, delete the `validatingwebhookconfiguration` manually by running
  `kubectl delete validatingwebhookconfiguration crossplane`.


#### <a id='1-10-0-config-server-ki'></a> v1.10.0 Known issues: Enterprise Config Server

- Enterprise Config Server is not supported on Openshift.

#### <a id='1-10-0-scanning-ki'></a> v1.10.0 Known issues: Scanning

- When uninstalling Tanzu Application Platform v1.10, the removal of the scanning package can
  get stuck because of a failed namespace deletion in the scanning package. You might see the
  following error:

    ```console
    Useful Error Message:  kapp: Error: Timed out waiting after 15m0s for resources: [namespace/metadata-store-secrets (v1) cluster]
    ```

    To work around this issue, do one of the following actions:

    - Remove the finalizer present on the namespace by running:

      ```console
      kubectl edit namespace/metadata-store-secrets
      ```

    - If a finalizer is not already present, add a finalizer by adding the following YAML with the
      `kubectl edit` command:

        ```yaml
        spec:
          finalizer:
          - kubernetes
        ```

#### <a id='1-10-0-stk-ki'></a> v1.10.0 Known issues: Services Toolkit

- An error occurs if `additionalProperties` is `true` in a CompositeResourceDefinition.
  For more information and a workaround, see [Troubleshoot Services Toolkit](./services-toolkit/how-to-guides/troubleshooting.hbs.md#compositeresourcedef).

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

- When you select the **Supply Chains** tab in Tanzu Developer Portal, you might encounter an error
  related to `data.packaging.carvel.dev`. The error message is related to permission issues and JSON
  parsing errors. The error message indicates that the user `system:serviceaccount:tap-gui:tap-gui-viewer`
  cannot list resource `packages` in the API group `data.packaging.carvel.dev` at the cluster scope.
  Additionally, an unexpected non-whitespace character is reported after JSON at position 4.

  As a temporary workaround, apply an RBAC configuration that includes the get, watch, and list
  permissions for the resources in the `data.packaging.carvel.dev` API group. This workaround must
  not be mandated for supply chains that do not generate Carvel packages.

  To eliminate the error message, configure RBAC to allow access to the Carvel package resource as
  follows:

   ```console
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRole
   - apiGroups: [data.packaging.carvel.dev]
     resources: [packages]
     verbs: ['get', 'watch', 'list']
   ```

#### <a id='1-10-0-supply-chain-ki'></a> v1.10.0 Known issues: Tanzu Supply Chain

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

#### <a id='1-10-0-scc-ki'></a> v1.10.0 Known issues: Supply Chain Choreographer

- The template for the `external-deliverable-template` does not respect the
  `gitops_credentials_secret` parameter. The value is not present on the deliverable if it is
  provided in the workload parameter `gitops_credentials_secret` or the supply chain tap-value
  `ootb_supply_chain*.gitops.credentials_secret`. As a workaround, operators must provide the value
  as a tap-value for the delivery: `ootb_delivery_basic.source.credentials_secret`.
  The supply chain's GitOps credentials must authenticate to the same repository as the delivery's
  source credentials. If a deliverable must use a secret different from that specified by the
  delivery tap-value, the deliverable must be manually altered when being copied to the Run
  cluster. Add the secret name as a `source_credentials_secret` parameter on the deliverable.

- By default, Server Workload Carvel packages generated by the Carvel package supply chains no longer
  contain OpenAPIv3 descriptions of their parameters.
  These descriptions were omitted to keep the size of the Carvel Package definition under 4&nbsp;KB,
  which is the size limit for the string output of a Tekton Task. For information about these parameters,
  see [Carvel Package Supply Chains](scc/carvel-package-supply-chain.hbs.md).

- When using the Carvel Package Supply Chains, if the operator updates the parameter
  `carvel_package.name_suffix`, existing workloads incorrectly output a Carvel package to the GitOps
  repository that uses the old value of `carvel_package.name_suffix`. You can ignore or delete this package.

- If the size of the resulting OpenAPIv3 specification exceeds a certain size, approximately 3&nbsp;KB,
  the Supply Chain does not function. If you use the default Carvel package parameters, this
  issue does not occur. If you use custom Carvel package parameters, you might encounter this size limit.
  If you exceed the size limit, you can either deactivate this feature, or use a workaround.
  The workaround requires enabling a Tekton feature flag. For more information, see the
  [Tekton documentation](https://tekton.dev/docs/pipelines/additional-configs/#enabling-larger-results-using-sidecar-logs).

- Supply Chains that use [SSH auth](scc/git-auth.hbs.md#sshassh) with the git-writer resource will fail in the
  [gitops](scc/gitops-vs-regops.hbs.md#gitopsagitops) step. As a workaround, use
  [HTTPS auth](scc/git-auth.hbs.md#httpahttp).

#### <a id='1-10-0-scst-policy-ki'></a> v1.10.0 Known issues: Supply Chain Security Tools - Policy

- Supply Chain Security Tools - Policy defaults to The Update Framework (TUF) enabled due to incorrect logic.
  This might cause the package to not reconcile correctly if the default TUF mirrors are not reachable.
  To work around this, explicitly configure policy controller in the `tap-values.yaml` file to
  enable TUF:

    ```yaml
    policy:
      tuf_enabled: true
    ```

#### <a id='1-10-0-scst-scan-ki'></a> v1.10.0 Known issues: Supply Chain Security Tools - Scan

- The Snyk scanner outputs an incorrectly created date, resulting in an invalid date. If the workload
  is in a failed state due to an invalid date, wait approximately 10 hours and the workload
  automatically goes into the ready state.
  For more about this issue information, see the
  [Snyk](https://github.com/snyk-tech-services/snyk2spdx/issues/54) GitHub repository.

- Recurring scan has a maximum of approximately 5000 container images that can be scanned at a
  single time due to size limits configMaps.

- If the supply chain container image scanning is configured to use a different scanner or scanner
  version than the recurring scanning, the vulnerabilities displayed in Tanzu Developer Portal might
  be inaccurate.

- SCST - Scan 1.0 fails with the error `secrets 'store-ca-cert' not found` during deployment by using
  Tanzu Mission Control with a non-default issuer. For how to work around this issue, see
  [Deployment failure with non-default issuer](scst-scan/troubleshoot-scan.hbs.md#non-default-issuer).

#### <a id='1-10-0-scst-scan-2-ki'></a> v1.10.0 Known issues: Supply Chain Security Tools - Scan 2.0

- When installing the Tanzu Application Platform Build profile or Full profile,
  Supply Chain Security Tools (SCST) - Scan 2.0 is also installed on the cluster.
  If you installed SCST - Scan 2.0 manually in an earlier Tanzu Application Platform version,
  uninstall SCST - Scan 2.0 before upgrading to v1.10 to avoid conflict.

#### <a id='1-10-0-scst-store-ki'></a> v1.10.0 Known issues: Supply Chain Security Tools - Store

- SCST - Store returns an expired certificate error message when a CA certificate expires before the
  app certificate.
  For more information, see [CA Cert expires](scst-store/troubleshooting.hbs.md#ca-cert-expires).

- When outputting CycloneDX v1.5 SBOMs, the report is found to be an invalid SBOM by CycloneDX validators.
  This issue is planned to be fixed in a future release.

- SCST - Store automatically detects PostgreSQL database index corruptions.
  If SCST - Store finds a PostgresSQL database index has been corrupted, SCST - Store will automatically
  attempt to repair, which might cause reconciliation during package updates. When this happens, the included Postgres database might take some time to complete the repair and accept connections. For more information, see [Fix Postgres Database Index Corruption](scst-store/database-index-corruption.hbs.md).

- If CA Certificate data is included in the shared Tanzu Application Platform values section, do not
  configure AMR Observer with CA Certificate data.

- When `observer.deploy_through_tmc` is `true`, properties are auto-configured for Tanzu Mission
  Control (TMC). This causes the `MultiClusterPropertyCollector` resource to overwrite existing
  Tanzu Application Platform values for Observer.

  When using Let's Encrypt ACME issuers, the resultant Kubernetes secret resource does
  not contain a `ca.crt` property. Therefore, when the `MultiClusterPropertyCollector` resource
  creates the Observer package configuration values secret, the required `ca_cert_data` is empty.

  To work around this issue, add the Certificate Authority (CA) Certificate to the
  `shared.ca_cert_data` key in the Tanzu Application Platform installation values.

#### <a id='1-10-0-tbs-ki'></a> v1.10.0 Known issues: Tanzu Build Service

- During upgrades a large number of builds might be created due to buildpack and stack updates.
  Some of these builds might fail due to transient network issues,
  causing the workload to be in an unhealthy state. This resolves itself on subsequent builds
  after a code change and does not affect the running application.

  If you do not want to wait for subsequent builds to run, you can manually trigger a build.
  For instructions, see [Troubleshooting](./tanzu-build-service/troubleshooting.hbs.md#failed-builds-upgrade).

#### <a id='1-10-0-tdp-ki'></a>v1.10.0 Known issues: Tanzu Developer Portal

- If you do not configure any authentication providers, and do not allow guest access, the following
  message appears when loading Tanzu Developer Portal in a browser:

    ```console
    No configured authentication providers. Please configure at least one.
    ```

    To resolve this issue, see [Troubleshooting](tap-gui/troubleshooting.hbs.md#authn-not-configured).

- Ad-blocking browser extensions and standalone ad-blocking software can interfere with telemetry
  collection within the VMware
  [Customer Experience Improvement Program](https://www.vmware.com/solutions/trustvmware/ceip.html)
  and restrict access to all or parts of Tanzu Developer Portal.
  For more information, see [Troubleshooting](tap-gui/troubleshooting.hbs.md#ad-block-interference).

- [ScmAuth](https://backstage.io/docs/reference/integration-react.scmauth/) is a Backstage concept
  that abstracts Source Code Management (SCM) authentication into a package. An oversight in a
  recent code-base migration led to the accidental exclusion of custom ScmAuth functions. This
  exclusion affected some client operations, such as using Application Accelerators to create Git
  repositories on behalf of users.

- The back-end Kubernetes plug-in reports failure in multicluster environments.
  In a multicluster environment when one request to a Kubernetes cluster fails,
  `backstage-kubernetes-backend` reports a failure to the front end.
  This is a known issue with upstream Backstage and it applies to all released versions of
  Tanzu Developer Portal. For more information, see
  [this Backstage code in GitHub](https://github.com/backstage/backstage/blob/c7f88d041b671185dc7a01e716f80dca0709e2a1/plugins/kubernetes-backend/src/service/KubernetesFanOutHandler.ts#L250-L271).
  This behavior arises from the API at the Backstage level. There are currently no known workarounds.
  There are plans for upstream commits to Backstage to resolve this issue.

- **Supply Chain UI plug-in:**

    On the **Workload Details** page, the config writer step takes longer
    than 20 seconds to load when more than 149 workloads (deployed in a single namespace) are
    displayed in the Supply Chain UI.

#### <a id='1-10-0-intellij-plugin-ki'></a> v1.10.0 Known issues: Tanzu Developer Tools for IntelliJ

- The error `com.vdurmont.semver4j.SemverException: Invalid version (no major version)` is shown in
  the error logs when attempting to perform a workload action before installing the Tanzu CLI apps
  plug-in.

- If you restart your computer while running Live Update without terminating the Tilt
  process beforehand, there is a lock that incorrectly shows that Live Update is still running and
  prevents it from starting again.
  For the fix, see [Troubleshooting](intellij-extension/troubleshooting.hbs.md#lock-prevents-live-update).

- Workload actions and Live Update do not work when in a project with spaces in its name, such as
  `my app`, or in its path, such as `C:\Users\My User\my-app`.
  For more information, see [Troubleshooting](intellij-extension/troubleshooting.hbs.md#projects-with-spaces).

- An **EDT Thread Exception** error is logged or reported as a notification with a message similar to
  `"com.intellij.diagnostic.PluginException: 2007 ms to call on EDT TanzuApplyAction#update@ProjectViewPopup"`.
  For more information, see
  [Troubleshooting](intellij-extension/troubleshooting.hbs.md#ui-liveness-check-error).

#### <a id='1-10-0-vs-plugin-ki'></a> v1.10.0 Known issues: Tanzu Developer Tools for Visual Studio

- Clicking the red square Stop button in the Visual Studio top toolbar can cause a workload to fail.
  For more information, see [Troubleshooting](vs-extension/troubleshooting.hbs.md#stop-button).

---

### <a id='1-10-0-components'></a> v1.10.0 Component versions

The following table lists the Tanzu Application Platform package versions included with this release.
For open source component versions in this Tanzu Application Platform release, see
[Open source component versions](oss-component-versions.hbs.md).

| Component Name                                     | Version                  |
| -------------------------------------------------- | ------------------------ |
| API Auto Registration                              | 0.5.0                    |
| API portal                                         | 1.5.0                    |
| Application Accelerator                            | 1.10.0                   |
| Application Configuration Service                  | 2.3.1                    |
| Application Live View APIServer                    | 1.10.0                   |
| Application Live View back end                     | 1.10.0                   |
| Application Live View connector                    | 1.10.0                   |
| Application Live View conventions                  | 1.10.0                   |
| Application Single Sign-On                         | 5.1.6                    |
| Artifact Metadata Repository Observer              | 0.6.0                    |
| AWS Services                                       | 0.4.0                    |
| Bitnami Services                                   | 0.6.0                    |
| Carbon Black Scanner for SCST - Scan (beta)        | 1.4.0                    |
| Cartographer Conventions                           | 0.9.1                    |
| cert-manager                                       | 2.9.0                    |
| Cloud Native Runtimes                              | 2.6.0                    |
| Contour                                            | 2.4.0                    |
| Crossplane                                         | 0.6.0                    |
| Default Roles                                      | 1.1.0                    |
| Developer Conventions                              | 0.16.1                   |
| Enterprise Config Server                           | 1.0.0                    |
| External Secrets Operator                          | 0.9.4+tanzu.3            |
| Flux CD Source Controller                          | 1.1.2+tanzu.4.1714385349 |
| Grype Scanner for SCST - Scan                      | 1.9.1                    |
| Local Source Proxy                                 | 0.2.1                    |
| Managed Resource Controller (beta)                 | 0.3.7                    |
| Namespace Provisioner                              | 0.6.2                    |
| Out of the Box Delivery - Basic                    | 0.16.9                   |
| Out of the Box Supply Chain - Basic                | 0.16.9                   |
| Out of the Box Supply Chain - Testing              | 0.16.9                   |
| Out of the Box Supply Chain - Testing and Scanning | 0.16.9                   |
| Out of the Box Templates                           | 0.16.9                   |
| Service Bindings                                   | 0.12.1                   |
| Service Registry                                   | 1.4.0                    |
| Services Toolkit                                   | 0.15.0                   |
| Snyk Scanner for SCST - Scan (beta)                | 1.3.0                    |
| Source Controller                                  | 0.9.1                    |
| Spring Boot conventions                            | 1.10.0                   |
| Spring Cloud Gateway                               | 2.2.4                    |
| Supply Chain Choreographer                         | 0.9.1                    |
| Supply Chain Security Tools - Policy Controller    | 1.6.4                    |
| Supply Chain Security Tools - Scan                 | 1.9.1                    |
| Supply Chain Security Tools - Scan 2.0             | 0.5.0                    |
| Supply Chain Security Tools - Store                | 1.10.0                   |
| Tanzu Application Platform Telemetry               | 0.7.0                    |
| Tanzu Build Service                                | 1.13.0                   |
| Tanzu CLI                                          | 1.3.0                    |
| Tanzu Developer Portal                             | 1.10.1                   |
| Tanzu Developer Portal Configurator                | 1.10.1                   |
| Tanzu Supply Chain (beta)                          | 0.3.8                    |
| Tekton Pipelines                                   | 0.50.3+tanzu.4           |

---

## <a id='deprecations'></a> Deprecations

The following features, listed by component, are deprecated.
Deprecated features remain on this list until they are retired from Tanzu Application Platform.

### <a id='cnrs-deprecations'></a> Cloud Native Runtimes deprecations

- **`default_tls_secret` config option**: This config option is now in `contour.default_tls_secret`
  and is marked for removal in Cloud Native Runtimes v2.7.
  In the meantime, both options are supported, and `contour.default_tls_secret` takes precedence over
  `default_tls_secret`.

- **`ingress.[internal/external].namespace` config options**: These config options are now in
  `contour.[internal/external].namespace` are marked for removal in Cloud Native Runtimes v2.7.
  In the meantime, both options are supported, and `contour.[internal/external].namespace` takes
  precedence over `ingress.[internal/external].namespace`.

### <a id='svc-toolkit-deprecations'></a> Services Toolkit deprecations

- The following APIs are deprecated and are marked for removal in Tanzu Application Platform v1.11:
  - `clusterexampleusages.services.apps.tanzu.vmware.com/v1alpha1`
  - `clusterresources.services.apps.tanzu.vmware.com/v1alpha1`

### <a id="sc-deprecations"></a> Source Controller deprecations

- The Source Controller `ImageRepository` API is deprecated and is marked for removal.
  Use the `OCIRepository` API instead. The Flux Source Controller installation includes the
  `OCIRepository` API. For more information about the `OCIRepository` API, see the
  [Flux documentation](https://fluxcd.io/flux/components/source/ocirepositories/).

### <a id='scst-scan-deprecations'></a> Supply Chain Security Tools - Scan v1.0 deprecation

- SCST - Scan v1.0 is deprecated, but it remains the default option for online installation. SCST -
  Scan v2.0 will be the default in Tanzu Application Platform v1.11. SCST - Scan v1.0 will be
  removed in a future Tanzu Application Platform version. For more information, see
  [SCST - Scan versions](scst-scan/overview.hbs.md#scst-scan-feat).

### <a id='scst-store-deprecations'></a> Supply Chain Security Tools - Store deprecations

- The [metadata-store (MDS)](scst-store/mds-overview.hbs.md) component within SCST - Store is
  deprecated and is marked for removal in a future Tanzu Application Platform version.

### <a id='tanzu-cli-deprecations'></a> Tanzu CLI deprecations

- The [Tanzu Insight plug-in](cli-plugins/insight/cli-overview.hbs.md) for the Tanzu CLI is
  deprecated and is marked for removal in a future Tanzu Application Platform version.

### <a id='tdp-deprecations'></a> Tanzu Developer Portal deprecations

- [Tanzu Developer Portal Configurator](tap-gui/configurator/about.hbs.md) is deprecated and is
  marked for removal in Tanzu Application Platform v1.11.

### <a id="tekton-deprecations"></a> Tekton Pipelines deprecations

- Tekton `ClusterTask` is deprecated and marked for removal. Use the `Task` API instead.
  For more information, see the [Tekton documentation](https://tekton.dev/docs/pipelines/deprecations/).
