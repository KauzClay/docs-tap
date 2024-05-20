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

- [SonarQube Scan Tanzu Supply Chain Component](scst-scan/tanzu-supply-chain/sonarqube-scan/overview.hbs.md):
  SonarQube Scan Tanzu Supply Chain Component is a new component that can be added to a Tanzu Supply Chain to perform Static Application Security Testing(SAST) scans
  on the source code configured. This component is in alpha at this time, and only supports scanning for maven projects.

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

- Supports L7 Routing to web workloads with TKGm and NSX ALB. For more information, see
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
<td>fluxcd.source.controller.tanzu.vmware.com</td>
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

  - Fixed performance issues in the Supply Chain UI that caused the Workloads page and Workload
    Details page to load slowly.

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

#### <a id='1-10-0-scanning-ki'></a> v1.10.0 Known issues: Scanning

When uninstalling Tanzu Application Platform v1.10, sometimes the removal of the scanning package
gets stuck because of a failed namespace deletion in the scanning package. You might see the
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

#### <a id='1-10-0-tdp-ki'></a> v1.10.0 Known issues: Tanzu Developer Portal

- **Supply Chain UI plug-in:** On the Workload Details page, the config writer step takes longer
  than 20 seconds to load when more than 149 workloads (deployed in a single namespace) are
  displayed in the Supply Chain UI.

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
