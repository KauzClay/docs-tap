# Tanzu Application Platform release notes

This topic contains release notes for Tanzu Application Platform v{{ vars.url_version }}.

## <a id='1-11-0'></a> v1.11.0

**Release Date**: 2 July 2024

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

- Adds enhancements to minimize traffic disruptions during upgrades due to the draining of
  nodes required when upgrading Kubernetes. For more information, see
  [Recommendations for upgrading Kubernetes](cloud-native-runtimes/how-to-guides/app-operators/upgrade-kubernetes.hbs.md).
  The following enhancements were added:

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

  You can visualize your Carvel package deployments details and URLs for both server and web-type
  workloads. Ensure that your deliverable name is the same as the workload name for deliverables to
  be visualized correctly in Supply Chain UI. For more information, see
  [Visualize Carvel package deployment details](tap-gui/plugins/scc-tap-gui.hbs.md#visualize-carvel-dep).

---

### <a id='1-11-0-breaking-changes'></a> v1.11.0 Breaking changes

This release includes the following changes, listed by component and area.

#### <a id='1-11-0-tap-bc'></a> v1.11.0 Breaking changes: Tanzu Application Platform

- Tanzu Application Platform releases have migrated from VMware Tanzu Network to the Broadcom
  Support Portal and Broadcom registry. Using VMware Tanzu Network to install or upgrade Tanzu
  Application Platform is no longer supported.

  Before you upgrade, you must move the Tanzu Application Platform images from the Broadcom registry
  `tanzu.packages.broadcom.com` to your own registry. Make sure you move the images to your
  container image registry as part of the instructions in
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
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2961">CVE-2024-2961</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28835">CVE-2024-28835</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28834">CVE-2024-28834</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28085">CVE-2024-28085</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24785">CVE-2024-24785</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24784">CVE-2024-24784</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24783">CVE-2024-24783</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45290">CVE-2023-45290</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45289">CVE-2023-45289</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-35116">CVE-2023-35116</a></li>
</ul></details></td>
</tr>
<tr>
<td>alm-catalog.component.apps.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-33655">CVE-2024-33655</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2961">CVE-2024-2961</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28835">CVE-2024-28835</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28834">CVE-2024-28834</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26633">CVE-2024-26633</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26631">CVE-2024-26631</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26598">CVE-2024-26598</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26597">CVE-2024-26597</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26591">CVE-2024-26591</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26589">CVE-2024-26589</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26586">CVE-2024-26586</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26581">CVE-2024-26581</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24860">CVE-2024-24860</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24788">CVE-2024-24788</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-23851">CVE-2024-23851</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-23850">CVE-2024-23850</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-22705">CVE-2024-22705</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52612">CVE-2023-52612</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52610">CVE-2023-52610</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52609">CVE-2023-52609</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52603">CVE-2023-52603</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52600">CVE-2023-52600</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52470">CVE-2023-52470</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52469">CVE-2023-52469</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52467">CVE-2023-52467</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52464">CVE-2023-52464</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52463">CVE-2023-52463</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52462">CVE-2023-52462</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52458">CVE-2023-52458</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52457">CVE-2023-52457</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52456">CVE-2023-52456</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52454">CVE-2023-52454</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52451">CVE-2023-52451</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52449">CVE-2023-52449</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52448">CVE-2023-52448</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52445">CVE-2023-52445</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52444">CVE-2023-52444</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52443">CVE-2023-52443</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52442">CVE-2023-52442</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52441">CVE-2023-52441</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52436">CVE-2023-52436</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52429">CVE-2023-52429</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52340">CVE-2023-52340</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-46838">CVE-2023-46838</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-3867">CVE-2023-3867</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-38431">CVE-2023-38431</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-38430">CVE-2023-38430</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-38427">CVE-2023-38427</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-32258">CVE-2023-32258</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-32254">CVE-2023-32254</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-24023">CVE-2023-24023</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-1194">CVE-2023-1194</a></li>
</ul></details></td>
</tr>
<tr>
<td>amr-observer.apps.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2961">CVE-2024-2961</a></li>
</ul></details></td>
</tr>
<tr>
<td>application-configuration-service.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-4v7x-pqxf-cx7m">GHSA-4v7x-pqxf-cx7m</a></li>
<li><a href="https://github.com/advisories/GHSA-2wrp-6fg6-hmc5">GHSA-2wrp-6fg6-hmc5</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2961">CVE-2024-2961</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26308">CVE-2024-26308</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-25710">CVE-2024-25710</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-21094">CVE-2024-21094</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-21068">CVE-2024-21068</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-21012">CVE-2024-21012</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-21011">CVE-2024-21011</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-39326">CVE-2023-39326</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-35116">CVE-2023-35116</a></li>
</ul></details></td>
</tr>
<tr>
<td>base-jammy-stack-lite.buildpacks.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-35845">CVE-2024-35845</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-35844">CVE-2024-35844</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-35830">CVE-2024-35830</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-35829">CVE-2024-35829</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-35828">CVE-2024-35828</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-35811">CVE-2024-35811</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-33602">CVE-2024-33602</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-33601">CVE-2024-33601</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-33600">CVE-2024-33600</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-33599">CVE-2024-33599</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-32465">CVE-2024-32465</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-32021">CVE-2024-32021</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-32020">CVE-2024-32020</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-32004">CVE-2024-32004</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-32002">CVE-2024-32002</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27436">CVE-2024-27436</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27432">CVE-2024-27432</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27431">CVE-2024-27431</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27419">CVE-2024-27419</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27417">CVE-2024-27417</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27416">CVE-2024-27416</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27415">CVE-2024-27415</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27414">CVE-2024-27414</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27413">CVE-2024-27413</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27412">CVE-2024-27412</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27410">CVE-2024-27410</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27405">CVE-2024-27405</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27403">CVE-2024-27403</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27390">CVE-2024-27390</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27388">CVE-2024-27388</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27078">CVE-2024-27078</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27077">CVE-2024-27077</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27076">CVE-2024-27076</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27075">CVE-2024-27075</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27074">CVE-2024-27074</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27073">CVE-2024-27073</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27065">CVE-2024-27065</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27054">CVE-2024-27054</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27053">CVE-2024-27053</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27052">CVE-2024-27052</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27051">CVE-2024-27051</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27047">CVE-2024-27047</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27046">CVE-2024-27046</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27045">CVE-2024-27045</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27044">CVE-2024-27044</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27043">CVE-2024-27043</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27039">CVE-2024-27039</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27038">CVE-2024-27038</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27037">CVE-2024-27037</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27034">CVE-2024-27034</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27030">CVE-2024-27030</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27028">CVE-2024-27028</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-27024">CVE-2024-27024</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26915">CVE-2024-26915</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26907">CVE-2024-26907</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26906">CVE-2024-26906</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26903">CVE-2024-26903</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26901">CVE-2024-26901</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26898">CVE-2024-26898</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26897">CVE-2024-26897</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26895">CVE-2024-26895</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26894">CVE-2024-26894</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26891">CVE-2024-26891</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26889">CVE-2024-26889</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26885">CVE-2024-26885</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26884">CVE-2024-26884</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26883">CVE-2024-26883</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26882">CVE-2024-26882</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26881">CVE-2024-26881</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26880">CVE-2024-26880</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26879">CVE-2024-26879</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26878">CVE-2024-26878</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26877">CVE-2024-26877</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26875">CVE-2024-26875</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26874">CVE-2024-26874</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26872">CVE-2024-26872</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26870">CVE-2024-26870</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26863">CVE-2024-26863</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26862">CVE-2024-26862</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26861">CVE-2024-26861</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26859">CVE-2024-26859</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26857">CVE-2024-26857</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26856">CVE-2024-26856</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26855">CVE-2024-26855</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26852">CVE-2024-26852</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26851">CVE-2024-26851</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26846">CVE-2024-26846</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26845">CVE-2024-26845</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26843">CVE-2024-26843</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26840">CVE-2024-26840</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26839">CVE-2024-26839</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26838">CVE-2024-26838</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26835">CVE-2024-26835</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26833">CVE-2024-26833</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26820">CVE-2024-26820</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26816">CVE-2024-26816</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26809">CVE-2024-26809</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26805">CVE-2024-26805</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26804">CVE-2024-26804</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26803">CVE-2024-26803</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26802">CVE-2024-26802</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26801">CVE-2024-26801</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26798">CVE-2024-26798</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26795">CVE-2024-26795</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26793">CVE-2024-26793</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26792">CVE-2024-26792</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26791">CVE-2024-26791</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26790">CVE-2024-26790</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26788">CVE-2024-26788</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26787">CVE-2024-26787</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26782">CVE-2024-26782</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26779">CVE-2024-26779</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26778">CVE-2024-26778</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26777">CVE-2024-26777</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26776">CVE-2024-26776</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26774">CVE-2024-26774</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26773">CVE-2024-26773</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26772">CVE-2024-26772</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26771">CVE-2024-26771</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26769">CVE-2024-26769</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26766">CVE-2024-26766</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26764">CVE-2024-26764</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26763">CVE-2024-26763</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26754">CVE-2024-26754</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26752">CVE-2024-26752</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26751">CVE-2024-26751</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26750">CVE-2024-26750</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26749">CVE-2024-26749</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26748">CVE-2024-26748</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26747">CVE-2024-26747</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26744">CVE-2024-26744</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26743">CVE-2024-26743</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26737">CVE-2024-26737</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26736">CVE-2024-26736</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26735">CVE-2024-26735</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26733">CVE-2024-26733</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26688">CVE-2024-26688</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26659">CVE-2024-26659</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26651">CVE-2024-26651</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26622">CVE-2024-26622</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26603">CVE-2024-26603</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26601">CVE-2024-26601</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26585">CVE-2024-26585</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26584">CVE-2024-26584</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26583">CVE-2024-26583</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-22099">CVE-2024-22099</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-21823">CVE-2024-21823</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-0841">CVE-2024-0841</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-7042">CVE-2023-7042</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-6270">CVE-2023-6270</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52662">CVE-2023-52662</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52656">CVE-2023-52656</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52652">CVE-2023-52652</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52650">CVE-2023-52650</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52645">CVE-2023-52645</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52644">CVE-2023-52644</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52641">CVE-2023-52641</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52640">CVE-2023-52640</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52620">CVE-2023-52620</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52530">CVE-2023-52530</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52497">CVE-2023-52497</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52447">CVE-2023-52447</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52434">CVE-2023-52434</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-47233">CVE-2023-47233</a></li>
</ul></details></td>
</tr>
<tr>
<td>buildservice.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-pvcr-v8j8-j5q3">GHSA-pvcr-v8j8-j5q3</a></li>
<li><a href="https://github.com/advisories/GHSA-hj3v-m684-v259">GHSA-hj3v-m684-v259</a></li>
<li><a href="https://github.com/advisories/GHSA-7f9x-gw85-8grf">GHSA-7f9x-gw85-8grf</a></li>
<li><a href="https://github.com/advisories/GHSA-45x7-px36-x8w8">GHSA-45x7-px36-x8w8</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2961">CVE-2024-2961</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28835">CVE-2024-28835</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28834">CVE-2024-28834</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28757">CVE-2024-28757</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28085">CVE-2024-28085</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24785">CVE-2024-24785</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24784">CVE-2024-24784</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24783">CVE-2024-24783</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-22365">CVE-2024-22365</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-0727">CVE-2024-0727</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-0567">CVE-2024-0567</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-0553">CVE-2024-0553</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-6237">CVE-2023-6237</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-6129">CVE-2023-6129</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-5678">CVE-2023-5678</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52425">CVE-2023-52425</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-4641">CVE-2023-4641</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45290">CVE-2023-45290</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45289">CVE-2023-45289</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45284">CVE-2023-45284</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-39326">CVE-2023-39326</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2022-3715">CVE-2022-3715</a></li>
</ul></details></td>
</tr>
<tr>
<td>carbonblack.scanning.apps.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-xw73-rw38-6vjc">GHSA-xw73-rw38-6vjc</a></li>
<li><a href="https://github.com/advisories/GHSA-rhh4-rh7c-7r5v">GHSA-rhh4-rh7c-7r5v</a></li>
<li><a href="https://github.com/advisories/GHSA-qppj-fm5r-hxr3">GHSA-qppj-fm5r-hxr3</a></li>
<li><a href="https://github.com/advisories/GHSA-jq35-85cj-fj4p">GHSA-jq35-85cj-fj4p</a></li>
<li><a href="https://github.com/advisories/GHSA-hpxr-w9w7-g4gv">GHSA-hpxr-w9w7-g4gv</a></li>
<li><a href="https://github.com/advisories/GHSA-9p26-698r-w4hx">GHSA-9p26-698r-w4hx</a></li>
<li><a href="https://github.com/advisories/GHSA-8r3f-844c-mc37">GHSA-8r3f-844c-mc37</a></li>
<li><a href="https://github.com/advisories/GHSA-7ww5-4wqc-m92c">GHSA-7ww5-4wqc-m92c</a></li>
<li><a href="https://github.com/advisories/GHSA-4v7x-pqxf-cx7m">GHSA-4v7x-pqxf-cx7m</a></li>
<li><a href="https://github.com/advisories/GHSA-45x7-px36-x8w8">GHSA-45x7-px36-x8w8</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-4741">CVE-2024-4741</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-4603">CVE-2024-4603</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-35329">CVE-2024-35329</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-35328">CVE-2024-35328</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-35326">CVE-2024-35326</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-35325">CVE-2024-35325</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-33602">CVE-2024-33602</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-33601">CVE-2024-33601</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-33600">CVE-2024-33600</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-33599">CVE-2024-33599</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2961">CVE-2024-2961</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28835">CVE-2024-28835</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28834">CVE-2024-28834</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28757">CVE-2024-28757</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28182">CVE-2024-28182</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28085">CVE-2024-28085</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26462">CVE-2024-26462</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26461">CVE-2024-26461</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2511">CVE-2024-2511</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24790">CVE-2024-24790</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24789">CVE-2024-24789</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24787">CVE-2024-24787</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2398">CVE-2024-2398</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2236">CVE-2024-2236</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-7008">CVE-2023-7008</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52425">CVE-2023-52425</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-50495">CVE-2023-50495</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45918">CVE-2023-45918</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45288">CVE-2023-45288</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-4039">CVE-2023-4039</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-39326">CVE-2023-39326</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-39319">CVE-2023-39319</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-39318">CVE-2023-39318</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-29383">CVE-2023-29383</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2022-4899">CVE-2022-4899</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2022-41409">CVE-2022-41409</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2022-40735">CVE-2022-40735</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2022-3715">CVE-2022-3715</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2022-3219">CVE-2022-3219</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2022-27943">CVE-2022-27943</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2021-46848">CVE-2021-46848</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2020-22916">CVE-2020-22916</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2016-2781">CVE-2016-2781</a></li>
</ul></details></td>
</tr>
<tr>
<td>cnrs.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2961">CVE-2024-2961</a></li>
</ul></details></td>
</tr>
<tr>
<td>config-server.spring.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-4v7x-pqxf-cx7m">GHSA-4v7x-pqxf-cx7m</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2961">CVE-2024-2961</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24788">CVE-2024-24788</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-21094">CVE-2024-21094</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-21068">CVE-2024-21068</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-21012">CVE-2024-21012</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-21011">CVE-2024-21011</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-35116">CVE-2023-35116</a></li>
</ul></details></td>
</tr>
<tr>
<td>crossplane.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24788">CVE-2024-24788</a></li>
</ul></details></td>
</tr>
<tr>
<td>dotnet-core-lite.buildpacks.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-x84c-p2g9-rqv9">GHSA-x84c-p2g9-rqv9</a></li>
<li><a href="https://github.com/advisories/GHSA-4v7x-pqxf-cx7m">GHSA-4v7x-pqxf-cx7m</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-32473">CVE-2024-32473</a></li>
</ul></details></td>
</tr>
<tr>
<td>managed-resource-controller.apps.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2961">CVE-2024-2961</a></li>
</ul></details></td>
</tr>
<tr>
<td>metadata-store.apps.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-8pgv-569h-w5rw">GHSA-8pgv-569h-w5rw</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2961">CVE-2024-2961</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-0985">CVE-2024-0985</a></li>
</ul></details></td>
</tr>
<tr>
<td>nodejs-lite.buildpacks.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-xw73-rw38-6vjc">GHSA-xw73-rw38-6vjc</a></li>
<li><a href="https://github.com/advisories/GHSA-mq39-4gv4-mvpx">GHSA-mq39-4gv4-mvpx</a></li>
<li><a href="https://github.com/advisories/GHSA-8r3f-844c-mc37">GHSA-8r3f-844c-mc37</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-29018">CVE-2024-29018</a></li>
</ul></details></td>
</tr>
<tr>
<td>ootb-templates.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-33655">CVE-2024-33655</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28835">CVE-2024-28835</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28834">CVE-2024-28834</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28085">CVE-2024-28085</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26633">CVE-2024-26633</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26631">CVE-2024-26631</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26598">CVE-2024-26598</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26597">CVE-2024-26597</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26591">CVE-2024-26591</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26589">CVE-2024-26589</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26586">CVE-2024-26586</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26581">CVE-2024-26581</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24860">CVE-2024-24860</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2398">CVE-2024-2398</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-23851">CVE-2024-23851</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-23850">CVE-2024-23850</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-22705">CVE-2024-22705</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52612">CVE-2023-52612</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52610">CVE-2023-52610</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52609">CVE-2023-52609</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52603">CVE-2023-52603</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52600">CVE-2023-52600</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52470">CVE-2023-52470</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52469">CVE-2023-52469</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52467">CVE-2023-52467</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52464">CVE-2023-52464</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52463">CVE-2023-52463</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52462">CVE-2023-52462</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52458">CVE-2023-52458</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52457">CVE-2023-52457</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52456">CVE-2023-52456</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52454">CVE-2023-52454</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52451">CVE-2023-52451</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52449">CVE-2023-52449</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52448">CVE-2023-52448</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52445">CVE-2023-52445</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52444">CVE-2023-52444</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52443">CVE-2023-52443</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52442">CVE-2023-52442</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52441">CVE-2023-52441</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52436">CVE-2023-52436</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52429">CVE-2023-52429</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52340">CVE-2023-52340</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-46838">CVE-2023-46838</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-3867">CVE-2023-3867</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-38431">CVE-2023-38431</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-38430">CVE-2023-38430</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-38427">CVE-2023-38427</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-32258">CVE-2023-32258</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-32254">CVE-2023-32254</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-24023">CVE-2023-24023</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-1194">CVE-2023-1194</a></li>
</ul></details></td>
</tr>
<tr>
<td>python-lite.buildpacks.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-xw73-rw38-6vjc">GHSA-xw73-rw38-6vjc</a></li>
<li><a href="https://github.com/advisories/GHSA-xmmx-7jpf-fx42">GHSA-xmmx-7jpf-fx42</a></li>
<li><a href="https://github.com/advisories/GHSA-v994-f8vw-g7j4">GHSA-v994-f8vw-g7j4</a></li>
<li><a href="https://github.com/advisories/GHSA-mq39-4gv4-mvpx">GHSA-mq39-4gv4-mvpx</a></li>
<li><a href="https://github.com/advisories/GHSA-jq35-85cj-fj4p">GHSA-jq35-85cj-fj4p</a></li>
<li><a href="https://github.com/advisories/GHSA-8r3f-844c-mc37">GHSA-8r3f-844c-mc37</a></li>
<li><a href="https://github.com/advisories/GHSA-6wrf-mxfj-pf5p">GHSA-6wrf-mxfj-pf5p</a></li>
<li><a href="https://github.com/advisories/GHSA-4v7x-pqxf-cx7m">GHSA-4v7x-pqxf-cx7m</a></li>
<li><a href="https://github.com/advisories/GHSA-3fwx-pjgw-3558">GHSA-3fwx-pjgw-3558</a></li>
<li><a href="https://github.com/advisories/GHSA-33pg-m6jh-5237">GHSA-33pg-m6jh-5237</a></li>
<li><a href="https://github.com/advisories/GHSA-2mm7-x5h6-5pvq">GHSA-2mm7-x5h6-5pvq</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-29018">CVE-2024-29018</a></li>
</ul></details></td>
</tr>
<tr>
<td>ruby-lite.buildpacks.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-4v7x-pqxf-cx7m">GHSA-4v7x-pqxf-cx7m</a></li>
</ul></details></td>
</tr>
<tr>
<td>source.component.apps.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-4v7x-pqxf-cx7m">GHSA-4v7x-pqxf-cx7m</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-34397">CVE-2024-34397</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-33602">CVE-2024-33602</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-33601">CVE-2024-33601</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-33600">CVE-2024-33600</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-33599">CVE-2024-33599</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-32465">CVE-2024-32465</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-32021">CVE-2024-32021</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-32020">CVE-2024-32020</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-32004">CVE-2024-32004</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-32002">CVE-2024-32002</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2961">CVE-2024-2961</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28835">CVE-2024-28835</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28834">CVE-2024-28834</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28182">CVE-2024-28182</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28085">CVE-2024-28085</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26920">CVE-2024-26920</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26916">CVE-2024-26916</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26910">CVE-2024-26910</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26829">CVE-2024-26829</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26826">CVE-2024-26826</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26825">CVE-2024-26825</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26808">CVE-2024-26808</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26722">CVE-2024-26722</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26720">CVE-2024-26720</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26717">CVE-2024-26717</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26715">CVE-2024-26715</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26712">CVE-2024-26712</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26707">CVE-2024-26707</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26704">CVE-2024-26704</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26702">CVE-2024-26702</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26698">CVE-2024-26698</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26697">CVE-2024-26697</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26696">CVE-2024-26696</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26695">CVE-2024-26695</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26689">CVE-2024-26689</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26685">CVE-2024-26685</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26684">CVE-2024-26684</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26679">CVE-2024-26679</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26676">CVE-2024-26676</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26675">CVE-2024-26675</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26673">CVE-2024-26673</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26671">CVE-2024-26671</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26668">CVE-2024-26668</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26665">CVE-2024-26665</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26664">CVE-2024-26664</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26663">CVE-2024-26663</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26660">CVE-2024-26660</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26645">CVE-2024-26645</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26644">CVE-2024-26644</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26641">CVE-2024-26641</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26640">CVE-2024-26640</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26636">CVE-2024-26636</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26635">CVE-2024-26635</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26633">CVE-2024-26633</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26631">CVE-2024-26631</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26627">CVE-2024-26627</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26625">CVE-2024-26625</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26622">CVE-2024-26622</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26615">CVE-2024-26615</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26614">CVE-2024-26614</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26610">CVE-2024-26610</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26608">CVE-2024-26608</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26606">CVE-2024-26606</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26602">CVE-2024-26602</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26600">CVE-2024-26600</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26598">CVE-2024-26598</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26597">CVE-2024-26597</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26594">CVE-2024-26594</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26593">CVE-2024-26593</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26592">CVE-2024-26592</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26591">CVE-2024-26591</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26589">CVE-2024-26589</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26586">CVE-2024-26586</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26581">CVE-2024-26581</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24860">CVE-2024-24860</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24790">CVE-2024-24790</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24789">CVE-2024-24789</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24787">CVE-2024-24787</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24785">CVE-2024-24785</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24784">CVE-2024-24784</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24783">CVE-2024-24783</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2398">CVE-2024-2398</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-23851">CVE-2024-23851</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-23850">CVE-2024-23850</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-23849">CVE-2024-23849</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-22705">CVE-2024-22705</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2201">CVE-2024-2201</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-1151">CVE-2024-1151</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52643">CVE-2023-52643</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52642">CVE-2023-52642</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52638">CVE-2023-52638</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52637">CVE-2023-52637</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52635">CVE-2023-52635</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52633">CVE-2023-52633</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52631">CVE-2023-52631</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52627">CVE-2023-52627</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52623">CVE-2023-52623</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52622">CVE-2023-52622</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52619">CVE-2023-52619</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52618">CVE-2023-52618</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52617">CVE-2023-52617</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52616">CVE-2023-52616</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52615">CVE-2023-52615</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52614">CVE-2023-52614</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52612">CVE-2023-52612</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52610">CVE-2023-52610</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52609">CVE-2023-52609</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52608">CVE-2023-52608</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52607">CVE-2023-52607</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52606">CVE-2023-52606</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52604">CVE-2023-52604</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52603">CVE-2023-52603</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52602">CVE-2023-52602</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52601">CVE-2023-52601</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52600">CVE-2023-52600</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52599">CVE-2023-52599</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52598">CVE-2023-52598</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52597">CVE-2023-52597</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52595">CVE-2023-52595</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52594">CVE-2023-52594</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52588">CVE-2023-52588</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52587">CVE-2023-52587</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52530">CVE-2023-52530</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52498">CVE-2023-52498</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52494">CVE-2023-52494</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52493">CVE-2023-52493</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52492">CVE-2023-52492</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52491">CVE-2023-52491</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52489">CVE-2023-52489</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52486">CVE-2023-52486</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52470">CVE-2023-52470</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52469">CVE-2023-52469</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52467">CVE-2023-52467</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52464">CVE-2023-52464</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52463">CVE-2023-52463</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52462">CVE-2023-52462</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52458">CVE-2023-52458</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52457">CVE-2023-52457</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52456">CVE-2023-52456</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52454">CVE-2023-52454</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52451">CVE-2023-52451</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52449">CVE-2023-52449</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52448">CVE-2023-52448</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52445">CVE-2023-52445</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52444">CVE-2023-52444</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52443">CVE-2023-52443</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52442">CVE-2023-52442</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52441">CVE-2023-52441</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52436">CVE-2023-52436</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52435">CVE-2023-52435</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52429">CVE-2023-52429</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52340">CVE-2023-52340</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-47233">CVE-2023-47233</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-46838">CVE-2023-46838</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45290">CVE-2023-45290</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45289">CVE-2023-45289</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45288">CVE-2023-45288</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-3867">CVE-2023-3867</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-38431">CVE-2023-38431</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-38430">CVE-2023-38430</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-38427">CVE-2023-38427</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-32258">CVE-2023-32258</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-32254">CVE-2023-32254</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-24023">CVE-2023-24023</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-1194">CVE-2023-1194</a></li>
</ul></details></td>
</tr>
<tr>
<td>supply-chain-catalog.apps.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24790">CVE-2024-24790</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24789">CVE-2024-24789</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24788">CVE-2024-24788</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24787">CVE-2024-24787</a></li>
</ul></details></td>
</tr>
<tr>
<td>supply-chain.apps.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-w2h3-vvvq-3m53">GHSA-w2h3-vvvq-3m53</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2961">CVE-2024-2961</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24788">CVE-2024-24788</a></li>
</ul></details></td>
</tr>
<tr>
<td>tekton.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://github.com/advisories/GHSA-mw99-9chc-xw7r">GHSA-mw99-9chc-xw7r</a></li>
<li><a href="https://github.com/advisories/GHSA-9763-4f94-gfch">GHSA-9763-4f94-gfch</a></li>
<li><a href="https://github.com/advisories/GHSA-7ww5-4wqc-m92c">GHSA-7ww5-4wqc-m92c</a></li>
<li><a href="https://github.com/advisories/GHSA-4v7x-pqxf-cx7m">GHSA-4v7x-pqxf-cx7m</a></li>
<li><a href="https://github.com/advisories/GHSA-45x7-px36-x8w8">GHSA-45x7-px36-x8w8</a></li>
<li><a href="https://github.com/advisories/GHSA-449p-3h89-pw88">GHSA-449p-3h89-pw88</a></li>
<li><a href="https://github.com/advisories/GHSA-2q89-485c-9j2x">GHSA-2q89-485c-9j2x</a></li>
<li><a href="https://github.com/advisories/GHSA-2c7c-3mj9-8fqh">GHSA-2c7c-3mj9-8fqh</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-34397">CVE-2024-34397</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2961">CVE-2024-2961</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28835">CVE-2024-28835</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28834">CVE-2024-28834</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28757">CVE-2024-28757</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28182">CVE-2024-28182</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-28085">CVE-2024-28085</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26920">CVE-2024-26920</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26916">CVE-2024-26916</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26910">CVE-2024-26910</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26829">CVE-2024-26829</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26826">CVE-2024-26826</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26825">CVE-2024-26825</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26808">CVE-2024-26808</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26722">CVE-2024-26722</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26720">CVE-2024-26720</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26717">CVE-2024-26717</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26715">CVE-2024-26715</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26712">CVE-2024-26712</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26707">CVE-2024-26707</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26704">CVE-2024-26704</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26702">CVE-2024-26702</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26698">CVE-2024-26698</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26697">CVE-2024-26697</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26696">CVE-2024-26696</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26695">CVE-2024-26695</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26689">CVE-2024-26689</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26685">CVE-2024-26685</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26684">CVE-2024-26684</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26679">CVE-2024-26679</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26676">CVE-2024-26676</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26675">CVE-2024-26675</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26673">CVE-2024-26673</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26671">CVE-2024-26671</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26668">CVE-2024-26668</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26665">CVE-2024-26665</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26664">CVE-2024-26664</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26663">CVE-2024-26663</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26660">CVE-2024-26660</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26645">CVE-2024-26645</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26644">CVE-2024-26644</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26641">CVE-2024-26641</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26640">CVE-2024-26640</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26636">CVE-2024-26636</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26635">CVE-2024-26635</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26633">CVE-2024-26633</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26631">CVE-2024-26631</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26627">CVE-2024-26627</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26625">CVE-2024-26625</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26622">CVE-2024-26622</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26615">CVE-2024-26615</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26614">CVE-2024-26614</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26610">CVE-2024-26610</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26608">CVE-2024-26608</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26606">CVE-2024-26606</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26602">CVE-2024-26602</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26600">CVE-2024-26600</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26598">CVE-2024-26598</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26597">CVE-2024-26597</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26594">CVE-2024-26594</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26593">CVE-2024-26593</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26592">CVE-2024-26592</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26591">CVE-2024-26591</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26589">CVE-2024-26589</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26586">CVE-2024-26586</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-26581">CVE-2024-26581</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24860">CVE-2024-24860</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24855">CVE-2024-24855</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24787">CVE-2024-24787</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24785">CVE-2024-24785</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24784">CVE-2024-24784</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-24783">CVE-2024-24783</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2398">CVE-2024-2398</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-23851">CVE-2024-23851</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-23850">CVE-2024-23850</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-23849">CVE-2024-23849</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-22705">CVE-2024-22705</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2201">CVE-2024-2201</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-1151">CVE-2024-1151</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-1086">CVE-2024-1086</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-1085">CVE-2024-1085</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-0727">CVE-2024-0727</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-0646">CVE-2024-0646</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-0641">CVE-2024-0641</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-0607">CVE-2024-0607</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-0565">CVE-2024-0565</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-0340">CVE-2024-0340</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-0193">CVE-2024-0193</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-6932">CVE-2023-6932</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-6931">CVE-2023-6931</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-6915">CVE-2023-6915</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-6817">CVE-2023-6817</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-6622">CVE-2023-6622</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-6606">CVE-2023-6606</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-6237">CVE-2023-6237</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-6176">CVE-2023-6176</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-6129">CVE-2023-6129</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-6121">CVE-2023-6121</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-6040">CVE-2023-6040</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-6039">CVE-2023-6039</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-5678">CVE-2023-5678</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52643">CVE-2023-52643</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52642">CVE-2023-52642</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52638">CVE-2023-52638</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52637">CVE-2023-52637</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52635">CVE-2023-52635</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52633">CVE-2023-52633</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52631">CVE-2023-52631</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52627">CVE-2023-52627</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52623">CVE-2023-52623</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52622">CVE-2023-52622</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52619">CVE-2023-52619</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52618">CVE-2023-52618</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52617">CVE-2023-52617</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52616">CVE-2023-52616</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52615">CVE-2023-52615</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52614">CVE-2023-52614</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52612">CVE-2023-52612</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52610">CVE-2023-52610</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52609">CVE-2023-52609</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52608">CVE-2023-52608</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52607">CVE-2023-52607</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52606">CVE-2023-52606</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52604">CVE-2023-52604</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52603">CVE-2023-52603</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52602">CVE-2023-52602</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52601">CVE-2023-52601</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52600">CVE-2023-52600</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52599">CVE-2023-52599</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52598">CVE-2023-52598</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52597">CVE-2023-52597</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52595">CVE-2023-52595</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52594">CVE-2023-52594</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52588">CVE-2023-52588</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52587">CVE-2023-52587</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52581">CVE-2023-52581</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52580">CVE-2023-52580</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52573">CVE-2023-52573</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52567">CVE-2023-52567</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52564">CVE-2023-52564</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52563">CVE-2023-52563</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52530">CVE-2023-52530</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52529">CVE-2023-52529</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52523">CVE-2023-52523</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52520">CVE-2023-52520</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52519">CVE-2023-52519</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52515">CVE-2023-52515</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52513">CVE-2023-52513</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52498">CVE-2023-52498</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52494">CVE-2023-52494</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52493">CVE-2023-52493</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52492">CVE-2023-52492</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52491">CVE-2023-52491</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52489">CVE-2023-52489</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52486">CVE-2023-52486</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52470">CVE-2023-52470</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52469">CVE-2023-52469</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52467">CVE-2023-52467</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52464">CVE-2023-52464</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52463">CVE-2023-52463</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52462">CVE-2023-52462</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52458">CVE-2023-52458</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52457">CVE-2023-52457</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52456">CVE-2023-52456</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52454">CVE-2023-52454</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52451">CVE-2023-52451</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52449">CVE-2023-52449</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52448">CVE-2023-52448</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52445">CVE-2023-52445</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52444">CVE-2023-52444</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52443">CVE-2023-52443</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52442">CVE-2023-52442</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52441">CVE-2023-52441</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52436">CVE-2023-52436</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52435">CVE-2023-52435</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52429">CVE-2023-52429</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52425">CVE-2023-52425</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-52340">CVE-2023-52340</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-51782">CVE-2023-51782</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-51781">CVE-2023-51781</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-51780">CVE-2023-51780</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-51779">CVE-2023-51779</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-47233">CVE-2023-47233</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-46862">CVE-2023-46862</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-46838">CVE-2023-46838</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-46813">CVE-2023-46813</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-4641">CVE-2023-4641</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-46343">CVE-2023-46343</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45290">CVE-2023-45290</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45289">CVE-2023-45289</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-45288">CVE-2023-45288</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-4134">CVE-2023-4134</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-39326">CVE-2023-39326</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-3867">CVE-2023-3867</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-38431">CVE-2023-38431</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-38430">CVE-2023-38430</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-38427">CVE-2023-38427</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-35827">CVE-2023-35827</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-34324">CVE-2023-34324</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-32258">CVE-2023-32258</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-32257">CVE-2023-32257</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-32254">CVE-2023-32254</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-32252">CVE-2023-32252</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-32250">CVE-2023-32250</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-32247">CVE-2023-32247</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-2953">CVE-2023-2953</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-24023">CVE-2023-24023</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-23000">CVE-2023-23000</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-22995">CVE-2023-22995</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-1732">CVE-2023-1732</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2023-1194">CVE-2023-1194</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2022-48065">CVE-2022-48065</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2022-48063">CVE-2022-48063</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2022-47695">CVE-2022-47695</a></li>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2022-3715">CVE-2022-3715</a></li>
</ul></details></td>
</tr>
<tr>
<td>trivy.app-scanning.component.apps.tanzu.vmware.com</td>
<td><details><summary>Expand to see the list</summary><ul>
<li><a href="https://nvd.nist.gov/vuln/detail/CVE-2024-2961">CVE-2024-2961</a></li>
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

#### <a id='1-11-0-tap-ki'></a> v1.11.0 Known issues: Tanzu Application Platform

- Upgrading from v1.8.x, v1.9.x, or v1.10.x to Tanzu Application Platform v1.11.0 might fail initially
  on the Build profile, but will reconcile automatically within few seconds. There is no current workaround.

---

### <a id='1-11-0-components'></a> v1.11.0 Component versions

The following table lists the Tanzu Application Platform package versions included with this release.
For open source component versions in this Tanzu Application Platform release, see
[Open source component versions](oss-component-versions.hbs.md).

| Component Name                                               | Version |
| ------------------------------------------------------------ | ------- |
| API Auto Registration                                        |         |
| API portal                                                   |         |
| Application Accelerator                                      |         |
| Application Configuration Service                            |         |
| Application Live View APIServer                              |         |
| Application Live View back end                               |         |
| Application Live View connector                              |         |
| Application Live View conventions                            |         |
| Application Single Sign-On                                   |         |
| Artifact Metadata Repository Observer                        |         |
| AWS Services                                                 |         |
| Bitnami Services                                             |         |
| Cartographer Conventions                                     |         |
| cert-manager                                                 |         |
| Cloud Native Runtimes                                        |         |
| Contour                                                      |         |
| Crossplane                                                   |         |
| Default Roles                                                |         |
| Developer Conventions                                        |         |
| Enterprise Config Server                                     |         |
| External Secrets Operator                                    |         |
| Flux CD Source Controller                                    |         |
| Grype Scanner for SCST - Scan                                |         |
| Local Source Proxy                                           |         |
| Managed Resource Controller (beta)                           |         |
| Namespace Provisioner                                        |         |
| Out of the Box Delivery - Basic                              |         |
| Out of the Box Supply Chain - Basic                          |         |
| Out of the Box Supply Chain - Testing                        |         |
| Out of the Box Supply Chain - Testing and Scanning           |         |
| Out of the Box Templates                                     |         |
| Service Bindings                                             |         |
| Service Registry                                             |         |
| Services Toolkit                                             |         |
| Snyk Scanner for SCST - Scan (beta)                          |         |
| Source Controller                                            |         |
| Spring Boot conventions                                      |         |
| Spring Cloud Gateway                                         |         |
| Supply Chain Choreographer                                   |         |
| Supply Chain Security Tools - Policy Controller (deprecated) |         |
| Supply Chain Security Tools - Scan (deprecated)              |         |
| Supply Chain Security Tools - Scan 2.0                       |         |
| Supply Chain Security Tools - Store                          |         |
| Tanzu Application Platform Telemetry                         |         |
| Tanzu Build Service                                          |         |
| Tanzu CLI                                                    |         |
| Tanzu Developer Portal                                       |         |
| Tanzu Supply Chain (beta)                                    |         |
| Tekton Pipelines                                             |         |

---

## <a id='deprecations'></a> Deprecations

The following features, listed by component, are deprecated.
Deprecated features remain on this list until they are retired from Tanzu Application Platform.

### <a id='cnrs-deprecations'></a> Cloud Native Runtimes deprecations

- **`default_tls_secret` config option**:

  This config option is now in `contour.default_tls_secret` and is marked for removal in Cloud
  Native Runtimes. In the meantime, both options are supported, and `contour.default_tls_secret`
  takes precedence over `default_tls_secret`.

- **`ingress.[internal/external].namespace` config options**:

  These config options are now in `contour.[internal/external].namespace` are marked for removal in
  Cloud Native Runtimes. In the meantime, both options are supported, and
  `contour.[internal/external].namespace` takes precedence over
  `ingress.[internal/external].namespace`.

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

- SCST - Scan v1.0 is deprecated, but it remains the default option for online installation.
  SCST - Scan v2.0 is the default in Tanzu Application Platform v1.11. SCST - Scan v1.0 will be
  removed in a future Tanzu Application Platform version. For more information, see
  [SCST - Scan versions](scst-scan/overview.hbs.md#scst-scan-feat).

### <a id='scst-store-deprecations'></a> Supply Chain Security Tools - Store deprecations

- The [Metadata Store (MDS)](scst-store/mds-overview.hbs.md) component within SCST - Store is
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
