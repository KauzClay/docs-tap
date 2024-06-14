# Upgrade compatibility and recommendations for Tanzu Application Platform

This topic gives you general information to help you with upgrading Tanzu Application Platform (commonly known as TAP).

## <a id="upgrade-paths"></a> Supported upgrade paths

<!-- Update in each release using https://confluence.eng.vmware.com/pages/viewpage.action?pageId=789919969#TanzuApplicationPlatform(TAP)-N-2Upgradesupportwithnew6weeksminorandLTS -->
<!-- Do not use a variable. Update version number each release so it's visible if this info becomes outdated.  -->

Tanzu Application Platform v1.6 supports upgrading from v1.5.x.

## <a id="support"></a> Compatibility

This section describes the compatibility considerations for Tanzu Application Platform v{{vars.url_version}}.

### <a id="k8s-matrix"></a> Kubernetes compatibility

For supported Kubernetes versions, see [Kubernetes version support for Tanzu Application Platform](../k8s-matrix.hbs.md).

You must use a kubectl version that is within one minor version difference of your cluster.
For example, a v1.30 client can communicate with v1.29, v1.30, and v1.31 control planes.
Using the latest compatible version of kubectl helps avoid unforeseen issues.

### <a id="tanzu-cli-matrix"></a> Tanzu CLI compatibility

Tanzu Application Platform v{{vars.url_version}} is compatible with Tanzu CLI v1.0.0 and later
and Tanzu CLI plug-ins for Tanzu Application Platform v{{vars.url_version}}.x.
The Tanzu CLI core v1.2 that is distributed with Tanzu Application Platform is forward and backward compatible
with all supported Tanzu Application Platform versions.

There is a group of Tanzu CLI plug-ins that extend the Tanzu CLI core with Tanzu Application Platform
specific features. You can install the plug-ins as a group with a single command.
Versioned releases of the Tanzu Application Platform specific plug-in group align to each supported
Tanzu Application Platform version. This makes it easy to switch between different versions of
Tanzu Application Platforms environments.

## <a id="recommendations"></a> Recommendations

- **Test the upgrade in a staging or development environment:**
  Use an environment that mirrors your production setup.
  This allows you to identify any potential issues or compatibility issues with your workloads before
  performing the upgrade in production.

- **Consider your tolerance for downtime per environment:**
  For example, sandbox might not need to scale out replicas of workloads or Tanzu Application Platform components.
  Use PodDisruptionBudgets only in environments that require high availability, such as production
  or higher level test environments. Only configure PodDisruptionBudgets if `minScale` or `replicas`
  are greater than `1`.

- **Consider your tolerance for downtime per Tanzu Application Platform profile:**
  You might have different downtime tolerances for each Tanzu Application Platform profile.
  Take time to understand these as one size does not fit all.

- **Consider the resource requirements for the upgraded Kubernetes cluster:**
  This includes CPU, memory, and storage. Ensure that your underlying infrastructure can support the
  new version and accommodate any changes in resource use from your workloads.
  You might want to preemptively scale out your nodes instead of letting autoscaling start.

- **Perform health checks on your cluster and workloads before starting the upgrade:**
  This helps to identify any issues that might impact the upgrade or the stability of your
  workloads post-upgrade. For more information, see:

  - [View cluster: Pre-Upgrade Cluster Health Check](#pre-upgrade-check-view)
  - [Build cluster: Pre-Upgrade Cluster Health Check](#pre-upgrade-check-build)
  - [Run cluster: Pre-Upgrade Cluster Health Check](#pre-upgrade-check-run)

- **Implement robust monitoring and observability tools:**
  Use these tools to track the health and performance of your Kubernetes cluster and workloads before,
  during, and after the upgrade. This helps you to detect any anomalies or performance degradation post-upgrade.

- **Consider the availability needs for workloads:**
  Tune them with the proper number of replicas and corresponding PodDisruptionBudget.
  Consider customizing the Tanzu Application Platform `SupplyChain` to stamp out a `PodDisruptionBudget`
  for workloads if needed. Knative workloads come with their own `PodDisruptionBudget` components.
  Production workloads running a server workload type must have a replica count of `2` or more to
  avoid downtime during scheduling of resources. Knative resources must have `minScale` set to `2` or
  more to avoid downtime.

- **Review compatibility of custom components:**
  Review any custom components you’ve added to the cluster for compatibility with Kubernetes.

- **Review the upgrade preparation checklist:**
  During your upgrade, follow the [Upgrade preparation checklist](#checklist).

### <a id="app-considerations"></a> Recommendations for apps

- **Ensure that all apps have a readiness or liveness probes:**
  This helps to avoid disruption during upgrades of Kubernetes and Tanzu Application Platform.

- **Consider using pod affinity rules:**
  This helps to spread out workloads across nodes and availability zones.

- **Use a pod disruption budget for workloads:**
  This helps to avoid downtime during upgrades, especially during Kubernetes upgrades.

- **Ensure that your app is protected against failure:**
  For example, the app uses a 12 factor app design and it can handle graceful shutdown.

- **Ensure that you have copies of business critical apps in a non-production cluster:**
  This cluster is where the upgrade can be tested before upgrading the cluster that contains the production
  instance of the app.

## <a id="checklist"></a> Upgrade preparation checklist

Complete the tasks in the following table before you upgrade Tanzu Application Platform.

<table>
<thead>
<tr>
  <th>Component</th>
  <th>Description</th>
</tr>
</thead>
<tbody>

<tr>
  <td>Upgrade Tanzu CLI</td>
  <td>Upgrade to the latest Tanzu CLI and plug-in group by following the instructions in
  <a href="../install-tanzu-cli.hbs.md#cli-and-plugin">Install the Tanzu CLI</a>.</td>
</tr>

<tr>
  <td>Upgrade Cluster Essentials</td>
  <td>Upgrade <a href="https://docs.vmware.com/en/Cluster-Essentials-for-VMware-Tanzu/index.html">Cluster Essentials</a>.
  If your cluster is maintained by Tanzu Mission Control, you do not need to install or update
  Cluster Essentials.</td>
</tr>

<tr>
  <td>Relocate the Tanzu Application Platform package images</td>
  <td>Relocate images from <code>tanzu.packages.broadcom.com</code> to your internal registry by following
  the instructions in <a href="../install-online/profile.hbs.md#relocate-images">Relocate images to a registry</a>.</td>
</tr>

<tr>
  <td>Relocate the full Tanzu Build Service dependencies</td>
  <td>(Optional) Relocate the full Tanzu Build Service dependencies to your internal registry by following
  the instructions in <a href="../install-offline/tbs-offline-install-deps.hbs.md">Install the Tanzu Build Service dependencies</a>. <br><br>
  This is not required if you only want the lite dependencies.</td>
</tr>

<tr>
  <td>Review Kubernetes cluster capacity</td>
  <td>Verify that you have enough spare CPU and memory capacity on your Run and Build cluster nodes
  for the upgrade. You need to schedule capacity for the new pods that are created during an upgrade.</td>
</tr>

<tr>
  <td>Review any custom overlays</td>
  <td>Check if there were any changes between versions for your custom overlays. For example, for the
  Supply Chain components. To pull a package after upgrading to view its contents and run a comparison,
  run: <br><br><pre>imgpkg pull -b $(kubectl get $PACKAGE_NAME < -n tap-install \
  -ojson | jq -r '.spec.fetch[0].imgpkgBundle.image') \
  -o /tmp/$PACKAGE_NAME && code /tmp/$PACKAGE_NAME</pre>
  If you’re using an overlay for Tanzu Developer Portal to surface additional Backstage plug-ins,
  re-run the steps to rebuild your customized Tanzu Developer Portal image by following the instructions in
  <a href="../tap-gui/configurator/building.hbs.md">Build your customized Tanzu Developer Portal with Configurator</a>.
  The re-built image will be referenced in the Tanzu Developer Portal overlay secret.
  Rebuilding the image is necessary because Tanzu Developer Portal includes a pinned version of Backstage
  and versions of node.
  <br><br>
  Take into account updates to Backstage versions. Review overlays, especially updates to API versions
  of Kubernetes resources targeted by each overlay.</td>
</tr>

<tr>
  <td>Review breaking changes for the Tanzu Application Platform release</td>
  <td>
  See <a href="../release-notes.hbs.md#{{vars.dash_version}}-0-breaking-changes">v{{vars.url_version}} Breaking changes</a>
  in the release notes.</td>
</tr>

<tr>
  <td>Plan your order of upgrade by cluster and then environment</td>
  <td>Recommended order to upgrade clusters:
  <ol><li>View cluster</li><li>Build clusters</li><li>Run clusters</li></ol>
  <br><br>
  Upgrade a sandbox or test environment first before upgrading subsequent environments.
  <br><br>
  You cannot jump upgrade versions n+2 until Tanzu Application Platform v1.7.x.
  For the list of versions you can upgrade to Tanzu Application Platform v{{vars.url_version}}, see
  <a href="#upgrade-paths"> Supported upgrade paths</a> earlier in this topic.</td>
</tr>

<tr>
  <td>Check the storage capacity of your container registry, such as Harbor or Artifactory, for sufficient storage</td>
  <td>New containers are created during an upgrade. This can increase the need for storage capacity.</td>
</tr>

<tr>
  <td>If you are using GitOps RI, tag your GitOps repository to maintain a point in time of your
  Tanzu Application Platform configuration</td>
  <td>Example tag commands:
  <br><br>
<pre>
git tag -a v1.x.x
git push origin v1.x.x
</pre>
  </td>
</tr>

</tbody>
</table>

## <a id="view"></a> View cluster checklists

### <a id="pre-upgrade-check-view"></a> Pre-upgrade cluster health check - View cluster

Complete the tasks in the following table before you upgrade the Tanzu Application Platform View cluster.

<table>
<thead>
<tr>
  <th>Item</th>
  <th>Description</th>
</tr>
</thead>
<tbody>

<tr>
  <td>Platform health monitoring</td>
  <td>Ensure that you have some form of monitoring in place to allow observability into your cluster
  during an upgrade. </td>
</tr>

<tr>
  <td>Check the health of each <code>packageRepository</code> and <code>PackageInstall</code> on the cluster</td>
  <td>Run:
<pre>
tanzu package repository list -A
tanzu package installed list -A
</pre>
  </td>
</tr>

<tr>
  <td>Check the health of all kapp apps on the cluster</td>
  <td>Run: <pre>kapp list -A</pre></td>
</tr>

<tr>
  <td>Check capacity of nodes on the cluster.
  <br><br>
  You might want to preemptively scale out your nodes if they are at a higher capacity instead of
  letting autoscaling start during an upgrade.</td>
  <td>Run: <pre>kubectl top nodes</pre></td>
</tr>

<tr>
  <td>Verify basic Tanzu Application Platform features on the cluster</td>
  <td><ul>
    <li>Test that Tanzu Developer Portal is working.</li>
    <li>Test that you can generate code from Accelerators.</li>
  </ul></td>
</tr>

<tr>
  <td>Check the health of pods on the cluster</td>
  <td>Run: <pre>kubectl get pods -A --field-selector=status.phase!=Running,status.phase!=Succeeded</pre></td>
</tr>

</tbody>
</table>

### <a id="tap-considerations-view"></a> Upgrade considerations for Kubernetes - View Cluster

The following table lists information about upgrading Kubernetes specific to the View cluster.

<table>
<thead>
<tr>
  <th>Item</th>
  <th>Description</th>
</tr>
</thead>
<tbody>

<tr>
  <td>Scale out Metadata Store pods for high availability</td>
  <td>Consider <a href="../scst-store/failover.hbs.md">scaling out Metadata Store</a> to have more
  than one replica to limit downtime. For example:
  <br><br>
  <pre>
  metadata_store:
    app_replicas: 3
  </pre>
  </td>
</tr>

<tr>
  <td>Create a <code>PodDisruptionBudget</code> for the Metadata Store app for high availability</td>
  <td>For example:
  <br><br>
  <pre>
  apiVersion: policy/v1
  kind: PodDisruptionBudget
  metadata:
    name: metadata-store-pdb
    namespace: metadata-store
  spec:
    minAvailable: 60%
    selector:
      matchLabels:
        app: metadata-store-app
  </pre>
  </td>
</tr>

<tr>
  <td>Configure Contour to be a DaemonSet to avoid downtime. If downtime is acceptable set to type deployment.
  <br><br>
  This is only applicable if you are upgrading from Tanzu Application Platform v1.6 or earlier.
  Contour is a deployment by default as of Tanzu Application Platform v1.7.</td>
  <td>For example:
  <br><br>
  <pre>
  contour:
    envoy:
      workload:
        type: daemonset
  </pre>
  </td>
</tr>

<tr>
  <td>Scale out Tanzu Developer Portal pods for high availability</td>
  <td>For example:
  <br><br>
  <pre>
  tap_gui:
    deployment:
      replicas: 3
  </pre>
  When Tanzu Developer Portal pods are scaled out, there is a known issue with generating Accelerators.
  You must apply the following overlay to the Tanzu Developer Portal package:
  <br><br>
  <pre>
  apiVersion: v1
  kind: Secret
  metadata:
    name: tap-gui-session-mgmt-affinity-secret
    namespace: tap-install
  stringData:
    tap-gui-session-mgmt-affinity.yml: |
      #@ load("@ytt:overlay", "overlay")

      #@overlay/match by=overlay.subset({"kind":"HTTPProxy","metadata":{"name":"tap-gui"}})
      ---
      spec:
        routes:
          #@overlay/match by=overlay.subset({"services": [{"name": "server"}]})
          - services: []
            #@overlay/match missing_ok=True
            loadBalancerPolicy:
              strategy: Cookie
  </pre>
  And add the following to the <code>tap-values.yaml</code> file:
  <br><br>
  <pre>
  package_overlays:
    - name: "tap-gui"
      secrets:
        - name: "tap-gui-session-mgmt-affinity-secret"
  </pre>
  </td>
</tr>

<tr>
  <td>Create a <code>PodDisruptionBudget</code> for Tanzu Developer Portal for high availability</td>
  <td>For example:
  <br><br>
  <pre>
  apiVersion: policy/v1
  kind: PodDisruptionBudget
  metadata:
    name: tap-gui-pdb
    namespace: tap-gui
  spec:
    minAvailable: 60%
    selector:
      matchLabels:
        app: backstage
  </pre>
  </td>
</tr>

</tbody>
</table>

### <a id="tap-considerations-view"></a> Upgrade considerations for Tanzu Application Platform - View Cluster

The following table lists information about upgrading Tanzu Application Platform specific to the View cluster.

<table>
<thead>
<tr>
  <th>Item</th>
  <th>Description</th>
</tr>
</thead>
<tbody>

<tr>
  <td>If upgrading from Tanzu Application Platform v1.6 or earlier, set up Artifact Metadata Repository (AMR)</td>
  <td>There are two new endpoints created on the view cluster with AMR. You might need to add them to
  your allowlist or have custom certificates configured with Tanzu Application Platform v1.7 and later.
    <ul>
      <li><code>amr-cloudevent-handler.DOMAIN</code></li>
      <li><code>amr-graphql.DOMAIN</code></li>
    </ul>
  <!-- Note: separate AMR upgrade instructions if opted in to beta version (tap-values review necessary) -->
  </td>
</tr>

<tr>
<td>What to expect during the upgrade on the View cluster</td>
<td><ul>
  <li>Tanzu Developer Portal might show a gray or missing status on the Supply Chain page.</li>
  <li>You might have issues with viewing pages. Consider doing a hard fresh for browsers.</li>
</ul></td>
</tr>

<tr>
  <td>If using GitOps RI, update the <code>version.yaml</code> file with the new
  Tanzu Application Platform version</td>
  <td>You can find the file at the following location:
  <code>/clusters/CLUSTER-NAME/cluster-config/config/tap-install/.tanzu-managed/version.yaml</code></td>
</tr>

<tr>
  <td>If using GitOps RI, run the <code>deploy.sh</code> script to pull in latest External Secret Operator
  version with Tanzu Application Platform</td>
  <td>You can find the script at the following location:
  <code>/clusters/CLUSTER-NAME/tanzu-sync/scripts/deploy.sh</code></td>
</tr>

<tr>
  <td>If using GitOps RI, make adjustments to your <code>tap-values.yaml</code> file if needed, and
  push the updated <code>version.yaml</code> and <code>tap-values.yaml</code> files to the Git repository</td>
  <td>After pushing the changes to Git, run these commands to sync the changes to the cluster and
  start the upgrade:
  <br><br>
<pre>
kctrl app kick -a sync -n tanzu-sync
kctrl app kick -a tap -n tap-install
</pre>
  </td>
</tr>

</tbody>
</table>

### <a id="tap-considerations-view"></a> Post upgrade checks - View cluster

Complete the tasks in the following table after you upgrade the Tanzu Application Platform View cluster.

<table>
<thead>
<tr>
  <th>Item</th>
  <th>Description</th>
</tr>
</thead>
<tbody>

<tr>
  <td>Check the health of each <code>packageRepository</code> and <code>PackageInstall</code> on the cluster.
  <br><br>
  They must now display the newest version of Tanzu Application Platform.</td>
  <td>Run:
<pre>
tanzu package repository list -A
tanzu package installed list -A
</pre>
  </td>
</tr>

<tr>
  <td>Check the health of all kapp apps on the cluster</td>
  <td>Run: <pre>kapp list -A</pre></td>
</tr>

<tr>
  <td>Check the health of pods on the cluster</td>
  <td>Run: <pre>kubectl get pods -A --field-selector=status.phase!=Running,status.phase!=Succeeded</pre></td>
</tr>

<tr>
  <td>Verify that Tanzu Developer Portal starts</td>
  <td>Log into Tanzu Developer Portal and verify that the basic functionality still works as expected.</td>
</tr>

</tbody>
</table>

## <a id="build"></a> Build cluster checklists

### <a id="pre-upgrade-check-build"></a> Pre-upgrade cluster health check - Build cluster

Complete the tasks in the following table before you upgrade the Tanzu Application Platform Build cluster.

<table>
<thead>
<tr>
  <th>Item</th>
  <th>Description</th>
</tr>
</thead>
<tbody>

<tr>
  <td>Platform health monitoring</td>
  <td>Ensure that you have some form of monitoring in place to allow observability into your cluster
  during an upgrade.
  </td>
</tr>

<tr>
  <td>Check the health of each <code>packageRepository</code> and <code>PackageInstall</code> on the cluster.
  <br><br>
  Fix any broken packages or repositories.</td>
  <td>Run:
<pre>
tanzu package repository list -A
tanzu package installed list -A
</pre>
  </td>
</tr>

<tr>
  <td>Check the health of all kapp apps on the cluster.
  <br><br>
  Fix any kapp apps that are in a bad state before upgrading.</td>
  <td>Run: <pre>kapp list -A</pre></td>
</tr>

<tr>
  <td>Check capacity of nodes on the cluster.
  <br><br>
  You might want to preemptively scale out your nodes if they are at a higher capacity instead of
  letting autoscaling start during an upgrade.</td>
  <td>Run: <pre>kubectl top nodes</pre></td>
</tr>

<tr>
  <td>Check the health of workloads on the cluster.
  <br><br>
  Consider fixing any workloads that are in a bad state before upgrading.</td>
  <td>Run: <pre>tanzu apps workload list -A</pre></td>
</tr>

<tr>
  <td>Check the health of pods on the cluster.
  <br><br>
  Consider fixing any pods that are in a bad state before upgrading.</td>
  <td>Run: <pre>kubectl get pods -A --field-selector=status.phase!=Running,status.phase!=Succeeded</pre></td>
</tr>

</tbody>
</table>

### <a id="tap-considerations-build"></a> Upgrade considerations for Kubernetes - Build Cluster

The following table lists information about upgrading Kubernetes specific to the Build cluster.

<table>
<thead>
<tr>
  <th>Item</th>
  <th>Description</th>
</tr>
</thead>
<tbody>

<tr>
  <td>Image cache on nodes will be reset</td>
  <td>As new nodes are spun up on a cluster, it might take longer to pull images as the cache has not
   been filled. This is only noticeable on the first pull of images.</td>
</tr>

<tr>
  <td>Consider pre-scaling the number of nodes</td>
  <td>Upgrading Kubernetes can trigger new pods, which take resources on the node.
  Node autoscaling will start up, which can take time and impact the window of the upgrade.
  Consider scaling capacity before an upgrade.</td>
</tr>

<tr>
  <td>Expect to lose logs on completed pods for workloads</td>
  <td>Expect to lose pod logs for the tests, builds and scans for a workload. Tanzu Developer Portal
  will continue to show the build history, but logs will be missing.</td>
</tr>

</tbody>
</table>

### <a id="tap-considerations-build"></a> Upgrade considerations for Tanzu Application Platform - Build Cluster

The following table lists information about upgrading Tanzu Application Platform specific to the Build cluster.

<table>
<thead>
<tr>
  <th>Item</th>
  <th>Description</th>
</tr>
</thead>
<tbody>

<tr>
  <td>Contour package is removed as of Tanzu Application Platform v1.7</td>
  <td>The contour package is deleted from the Build Cluster as of Tanzu Application Platform v1.7.</td>
</tr>

<tr>
  <td>New versions of Buildpacks from the Tanzu Application Platform upgrade triggers new
  Tanzu Build Service builds for all workloads</td>
  <td>When upgrading Tanzu Application Platform, new buildpacks will be introduced.
  This triggers new Tanzu Build Service builds. Expect workloads affected by updated buildpacks to rebuild.
  Node autoscaling will happen.
  <br><br>
  Your container image registry might have bursts of traffic and activity with new builds.
  Plan accordingly.</td>
</tr>

<tr>
  <td>Set an overlay for resource limits on Kpack to optimize resources used by builds</td>
  <td>When new buildpacks are applied during an upgrade, expect all workloads to trigger a new build.
  This might flood your cluster with build requests. Ensure that you have scaled out enough nodes and
  the appropriate processors to avoid being starved for CPU or memory.
  </td>
</tr>

<tr>
  <td>If upgrading from TAP 1.6 or earlier, set up Artifact Metadata Repository (AMR) Observer on
  the Build cluster</td>
  <td>Set up the AMR Observer on the Build cluster by following the instructions in
  <a href="../scst-store/multicluster-setup.hbs.md">Multicluster setup for Supply Chain Security Tools - Store</a>.
  This communicates with the AMR EventHandler on the View cluster. For example:
  <br><br>
  Add the following to the Build cluster <code>tap-values.yaml</code> file:
  <br><br>
  <pre>
  amr:
    observer:
      auth:
        kubernetes_service_accounts:
          autoconfigure: false
          enable: true
      cloudevent_handler:
        endpoint: https://amr-cloudevent-handler.DOMAIN
      ca_cert_data: |
        -----BEGIN CERT------
  </pre>
  Run the following commands on the View cluster to retrieve the AMR CA certificate and access token.
  Apply the CA certificate and access token on the Build cluster.
  The CA Cert goes into the <code>tap-values.yaml</code> file. Add the edit token as a
  Kubernetes secret on the Build cluster.
  <br><br>
<pre>
CEH_CA_CERT_DATA=$(kubectl get secret amr-cloudevent-handler-ingress-cert \
  -n metadata-store -o json | jq -r ".data.\"tls.crt\"" | base64 -d)

CEH_EDIT_TOKEN=$(kubectl get secrets amr-cloudevent-handler-edit-token \
  -n metadata-store -o jsonpath="{.data.token}" | base64 -d)
</pre>
  Example external secret setup:
  <br><br>
  <pre>
  ---
  apiVersion: external-secrets.io/v1beta1
  kind: ExternalSecret
  metadata:
    name: amr-observer-edit-token
    namespace: tap-install
  spec:
    secretStoreRef:
      name: tap-install-secrets
      kind: SecretStore
    refreshInterval: "1m"
    target:
      template:
        data:
          token: "\{{ .amr_token }}"
    data:
      - secretKey: amr_token
        remoteRef:
          key: PATH-TO-EXTERNAL-SECRET-STORE
          property: amr_token

  ---
  apiVersion: secretgen.carvel.dev/v1alpha1
  kind: SecretExport
  metadata:
    name: amr-observer-edit-token
    namespace: tap-install
  spec:
    toNamespaces:
      - amr-observer-system
  ---
  apiVersion: secretgen.carvel.dev/v1alpha1
  kind: SecretImport
  metadata:
    name: amr-observer-edit-token
    namespace: amr-observer-system
  spec:
    fromNamespace: tap-install
  </pre>
  </td>
</tr>

<tr>
  <td>Update <code>SupplyChain</code> customizations for any changes</td>
  <td><code>ClusterTasks</code> were removed as of Tanzu Application Platform v1.7. For example,
  if you wrote your own <code>ClusterTemplate</code> that uses the <code>git-writer</code> task,
  you must reference it as follows:
  <br><br>
  <pre>
  taskRef:
    resolver: cluster
    params:
      - name: kind
        value: task
      - name: namespace
        value: tap-tasks
      - name: name
        value: git-writer
  </pre>
  If you’ve customized the <code>config-writer-template</code> in the <code>ClusterTemplate</code>,
  Ensure that the <code>configPath</code> property refers to <code>.spec.params</code> instead of
  <code>.spec.input.params</code>.</td>
</tr>

<tr>
  <td>Upgrading to Supply Chain Security Tools (SCST) - Scan 2.0 (SCST - Scan 1.0 is deprecated as of
  Tanzu Application Platform v1.10).
  <br><br>Trivy is the new default scanner.</td>
  <td>Upgrading from SCST - Scan 1.0 to SCST - Scan 2.0 in Tanzu Application Platform requires you to
  coordinate multiple steps across the Build and View clusters. There will be new token or certificate
  values to pull from the View cluster and configuration on the supply chain to move to Scan 2.0.
  <br><br>
  The scanning section on the Build cluster now needs the CA and Auth Token from the View cluster for
  Metadata store. If using GitOps RI, consider storing these in an external secret for sensitive values.
  <br><br>
  <pre>
  scanning:
    metadataStore:
      exports:
        ca:
          pem: |
            CONTENTS-OF-MDS-CA-CERT
        auth:
          token: CONTENTS-OF-MDS-AUTH-TOKEN
  </pre>
  Where <code>CONTENTS-OF-MDS-CA-CERT</code> is the contents of <code>$MDS_CA_CERT</code> and
  <code>CONTENTS-OF-MDS-AUTH-TOKEN</code> is the contet of <code>$MDS_AUTH_TOKEN</code>.
  <br><br>
  You can remove the Grype section from your <code>tap-values.yaml</code> file for the Build cluster.
  Trivy is now the default scanner for Tanzu Application Platform.
  <br><br>
  Ensure that you delete any configured secrets in the <code>metadata-store-secrets</code> namespace
  on the build cluster.
  These are configured automatically, so removing them helps to avoid issues like kapp ownership conflicts.
  <br><br>
  Configure Trivy Scanner in your <code>tap-values.yaml</code> file on the Build cluster.
  <br><br>
  <pre>
  ootb_supply_chain_testing_scanning:
        image_scanner_template_name: image-vulnerability-scan-trivy
        image_scanning_cli:
          image: PATH-TO-RELOCATED-IMAGE
  </pre>
  After applying the changes to the Build cluster and the reconciliation is successful, run the
  Namespace Provisioner on the Build cluster to reconcile any changes to each namespace.
  <br><br>
  Note: If you are moving away from Grype, set the following flag to <a href="../namespace-provisioner/use-case4.hbs.md#deactivate-grype-for-all">deactivate Grype for all namespaces</a>:
  <br><br>
  <pre>
  namespace_provisioner:
    default_parameters:
      skip_grype: true
  </pre>
  Note: If you use or you switch between different scanners in Tanzu Application Platform v1.9 and earlier,
  Metadata Store will show duplicate results in the Tanzu Developer Portal <strong>Security Analysis</strong>
  UI.
  </td>
</tr>

<tr>
  <td>If using GitOps RI, update the <code>version.yaml</code> file with the new
  Tanzu Application Platform version</td>
  <td>You can find the file at the following location:
  <code>/clusters/CLUSTER-NAME/cluster-config/config/tap-install/.tanzu-managed/version.yaml</code></td>
</tr>

<tr>
  <td>If using GitOps RI, run the <code>deploy.sh</code> script to pull in latest External Secret Operator
  version with Tanzu Application Platform</td>
  <td>You can find the script at the following location:
  <code>/clusters/CLUSTER-NAME/tanzu-sync/scripts/deploy.sh</code></td>
</tr>

<tr>
  <td>If using GitOps RI, make adjustments to your <code>tap-values.yaml</code> file if needed, and
  push the updated <code>version.yaml</code> and <code>tap-values.yaml</code> files to the Git repository</td>
  <td>After pushing the changes to Git, run the these commands to sync changes to the cluster and
  start the upgrade:
  <br><br>
<pre>
kctrl app kick -a sync -n tanzu-sync
kctrl app kick -a tap -n tap-install
</pre>
  </td>
</tr>

</tbody>
</table>

### <a id="tap-considerations-view"></a> Post upgrade checks - Build cluster

Complete the tasks in the following table after you upgrade the Tanzu Application Platform Build cluster.

<table>
<thead>
<tr>
  <th>Item</th>
  <th>Description</th>
</tr>
</thead>
<tbody>

<tr>
  <td>Check the health of each <code>packageRepository</code> and <code>PackageInstall</code> on the cluster.
  <br><br>
  They must now display the newest version of Tanzu Application Platform.</td>
  <td>Run:
<pre>
tanzu package repository list -A
tanzu package installed list -A
</pre>
  </td>
</tr>

<tr>
  <td>Check the health of all kapp apps on the cluster</td>
  <td>Run: <pre>kapp list -A</pre></td>
</tr>

<tr>
  <td>Check the health of pods on the cluster</td>
  <td>Run: <pre>kubectl get pods -A --field-selector=status.phase!=Running,status.phase!=Succeeded</pre></td>
</tr>

<tr>
  <td>Check the health of workloads on the cluster.
  <br><br>
  Consider fixing any workloads that are in a bad state before upgrading.</td>
  <td>Run: <pre>tanzu apps workload list -A</pre></td>
</tr>

<tr>
  <td>Check for workload scan results in Tanzu Developer Portal and that you can download the
  workload SBOM in supported formats</td>
  <td><em>n/a</em></td>
</tr>

</tbody>
</table>

## <a id="run"></a> Run cluster checklists

### <a id="pre-upgrade-check-run"></a> Pre-upgrade cluster health check - Run cluster

Complete the tasks in the following table before you upgrade the Tanzu Application Platform Run cluster.

<table>
<thead>
<tr>
  <th>Item</th>
  <th>Description</th>
</tr>
</thead>
<tbody>

<tr>
  <td>Platform health monitoring</td>
  <td>Ensure that you have some form of monitoring in place to allow observability into your cluster
  during an upgrade. </td>
</tr>

<tr>
  <td>Check the health of each <code>packageRepository</code> and <code>PackageInstall</code> on the cluster</td>
  <td>Run:
<pre>
tanzu package repository list -A
tanzu package installed list -A
</pre>
  </td>
</tr>

<tr>
  <td>Check the health of all kapp apps on the cluster</td>
  <td>Run: <pre>kapp list -A</pre></td>
</tr>

<tr>
  <td>Check capacity of nodes on the cluster</td>
  <td>Run: <pre>kubectl top nodes</pre></td>
</tr>

<tr>
  <td>Check the health of pods on the cluster</td>
  <td>Run: <pre>kubectl get pods -A --field-selector=status.phase!=Running,status.phase!=Succeeded</pre></td>
</tr>

</tbody>
</table>

### <a id="tap-considerations-run"></a> Upgrade considerations for Kubernetes - Run Cluster

The following table lists information about upgrading Kubernetes specific to the Run cluster.

<table>
<thead>
<tr>
  <th>Item</th>
  <th>Description</th>
</tr>
</thead>
<tbody>

<tr>
  <td>Configure Contour to be a DaemonSet to avoid downtime. If downtime is acceptable set to type deployment.
  <br><br>
  This is only applicable if you are upgrading from Tanzu Application Platform v1.6 or earlier.
  Contour is a deployment by default as of Tanzu Application Platform v1.7.</td>
  <td>For example:
  <br><br>
  <pre>
  contour:
    envoy:
      workload:
        type: daemonset
  </pre>
  </td>
</tr>

<tr>
  <td>Scale out your <code>AuthServer</code> and use a <code>PodDisruptionBudget</code> to ensure uptime</td>
  <td>Consider scaling out your <code>AuthServer</code> to multiple replicas and using a
  <code>PodDisruptionBudget</code> to ensure uptime.
  <br><br>
  Example <code>AuthServer</code>:
  <br><br>
  <pre>
  apiVersion: sso.apps.tanzu.vmware.com/v1alpha1
  kind: AuthServer
  ...
  spec:
    replicas: 3
  </pre>
  Example <code>PodDisruptionBudget</code>:
  <br><br>
  <pre>
  apiVersion: policy/v1
  kind: PodDisruptionBudget
  metadata:
    name: auth0-authserver
    namespace: shared-services
  spec:
    minAvailable: 50%
    selector:
      matchLabels:
        app.kubernetes.io/part-of: auth0-authserver
  </pre>
  </td>
</tr>

<tr>
  <td>Scale out Application Configuration Service and use a <code>PodDisruptionBudget</code> to ensure uptime</td>
  <td>Add replicas to the <code>PackageInstall</code> of Application Configuration Service and a
  <code>PodDisruptionBudget</code>.
  Example <code>PackageInstall</code>:
  <br><br>
  <pre>
  apiVersion: packaging.carvel.dev/v1alpha1
  kind: PackageInstall
  metadata:
    name: acs
    namespace: tap-install
  spec:
    packageRef:
      refName: application-configuration-service.tanzu.vmware.com
      versionSelection:
        constraints: ">=2.1.4"
        prereleases: {}
    serviceAccountName: tap-installer-sa
    values:
      - secretRef:
          name: acs-values
  ---
  apiVersion: v1
  kind: Secret
  metadata:
    name: acs-values
    namespace: tap-install
  stringData:
    values.yaml: |
      reconciler:
        replicas: 3
  </pre>
  Example <code>PodDisruptionBudget</code>:
  <br><br>
  <pre>
  apiVersion: policy/v1
  kind: PodDisruptionBudget
  metadata:
    name: acs
    namespace: application-configuration-service
  spec:
    minAvailable: 60%
    selector:
      matchLabels:
        app: application-configuration-service
  </pre>
  </td>
</tr>

<tr>
  <td>Consider Bitnami Charts replicas and pod disruption budgets for when running services inside
  the same cluster</td>
  <td>While Bitnami Charts are not intended for production, if you are using them in that
  manner then consider replicas and pod disruption budgets for each component.</td>
</tr>

<tr>
  <td>Consider replicas and pod disruption budgets for workloads.
  <br><br>
  You can use an overlay to create them as part of the <code>SupplyChain</code> by default.</td>
  <td>Every workload on Tanzu Application Platform must take replica and disruption budgets into
  consideration. Consider adding logic to the supply chain to create them for each component.
  <br><br>
  Example overlay on ootb-templates package install:
  <br><br>
  <pre>
  apiVersion: v1
  kind: Secret
  metadata:
    name: ootb-templates-overlay-pdb
    namespace: tap-install
    annotations:
      kapp.k14s.io/change-group: "tap-overlays"
  type: Opaque
  stringData:
    overlay-pdb.yml: |
      #@ load("@ytt:overlay", "overlay")
      #@ load("@ytt:data", "data")
      #@overlay/match by=overlay.subset({"kind":"ClusterConfigTemplate", "metadata": {"name": "config-template"}})
      ---
      spec:
        #@overlay/replace via=lambda left, right: "{}\n{}".format(left, '\n'.join(['  {}'.format(x) for x in right.split('\n')]))
        ytt: |
          #@yaml/text-templated-strings
          pdb.yaml: |
            (@ if hasattr(data.values.params, "annotations") and hasattr(data.values.params.annotations, "autoscaling.knative.dev/minScale") and int(getattr(data.values.params.annotations, "autoscaling.knative.dev/minScale")) > 1 : @)
            ---
            apiVersion: policy/v1
            kind: PodDisruptionBudget
            metadata:
              name: (@= data.values.workload.metadata.name @)
            spec:
              maxUnavailable: 1
              selector:
                matchLabels:
                  app.kubernetes.io/part-of: (@= data.values.workload.metadata.name @)
                  app.kubernetes.io/component: run
            (@ end @)
  </pre>
  </td>
</tr>

</tbody>
</table>

### <a id="tap-considerations-run"></a> Upgrade considerations for Tanzu Application Platform - Run Cluster

The following table lists information about upgrading Tanzu Application Platform specific to the Run cluster.

<table>
<thead>
<tr>
  <th>Item</th>
  <th>Description</th>
</tr>
</thead>
<tbody>

<tr>
  <td>Update replicas of the Application Live View connector and add a <code>PodDisruptionBudget</code>
  to ensure high availability.</td>
  <td>Update the Run cluster Application Live View connector to have more replicas:
  <br><br>
  <pre>
  appliveview_connector:
    connector:
      deployment:
        enabled: true
        replicas: 3
  </pre>
  Add a <code>PodDisruptionBudget</code> for the Application Live View Connector onto the cluster:
  <br><br>
  <pre>
  apiVersion: policy/v1
  kind: PodDisruptionBudget
  metadata:
    name: alv-connector
    namespace: app-live-view-connector
  spec:
    minAvailable: 60%
    selector:
      matchLabels:
        name: application-live-view-connector
  </pre>
  </td>
</tr>

<tr>
  <td>Set up Artifact Metadata Repository (AMR) Observer on the Run cluster</td>
  <td>Set up the AMR Observer on the Run cluster by following the instructions in
  <a href="../scst-store/multicluster-setup.hbs.md">Multicluster setup for Supply Chain Security Tools - Store</a>.
  This communicates with the AMR EventHandler on the View cluster. For example:
  <br><br>
  Add the following to the Run cluster <code>tap-values.yaml</code> file:
  <br><br>
  <pre>
  amr:
    observer:
      auth:
        kubernetes_service_accounts:
          autoconfigure: false
          enable: true
      cloudevent_handler:
        endpoint: https://amr-cloudevent-handler.DOMAIN
      ca_cert_data: |
        -----BEGIN CERT------
  </pre>
  Run the following commands on the View cluster to retrieve the AMR CA certificate and access token.
  Apply the CA certificate and access token on the Run cluster.
  The CA Cert goes into the <code>tap-values.yaml</code> file. Add the edit token as a Kubernetes
  secret on the Run cluster.
  <br><br>
  <pre>
CEH_CA_CERT_DATA=$(kubectl get secret amr-cloudevent-handler-ingress-cert \
  -n metadata-store -o json | jq -r ".data.\"tls.crt\"" | base64 -d)

CEH_EDIT_TOKEN=$(kubectl get secrets amr-cloudevent-handler-edit-token \
  -n metadata-store -o jsonpath="{.data.token}" | base64 -d)
</pre>
  </td>
</tr>

<tr>
  <td>Review any customizations to the <code>ClusterDelivery</code> or OOTB Templates</td>
  <td><em>n/a</em></td>
</tr>

<tr>
  <td>If using GitOps RI, update the <code>version.yaml</code> file with the new
  Tanzu Application Platform version</td>
  <td>You can find the file at the following location:
  <code>/clusters/CLUSTER-NAME/cluster-config/config/tap-install/.tanzu-managed/version.yaml</code></td>
</tr>

<tr>
  <td>If using GitOps RI, run the <code>deploy.sh</code> script to pull in latest
  External Secret Operator version with Tanzu Application Platform</td>
  <td>You can find the script at the following location:
  <code>/clusters/CLUSTER-NAME/tanzu-sync/scripts/deploy.sh</td>
</tr>

<tr>
  <td>If using GitOps RI, make adjustments to your <code>tap-values.yaml</code> file if needed,
  and push the updated <code>version.yaml</code> and <code>tap-values.yaml</code> files to the Git repository.</td>
  <td>After pushing the changes to Git, run these commands to sync changes to the cluster and
  start the upgrade:
  <br><br>
<pre>
kctrl app kick -a sync -n tanzu-sync
kctrl app kick -a tap -n tap-install
</pre>
  </td>
</tr>

</tbody>
</table>

### <a id="tap-considerations-view"></a> Post upgrade checks - Run cluster

Complete the tasks in the following table after you upgrade the Tanzu Application Platform Run cluster.

<table>
<thead>
<tr>
  <th>Item</th>
  <th>Description</th>
</tr>
</thead>
<tbody>

<tr>
  <td>Check the health of each <code>packageRepository</code> and <code>PackageInstall</code> on the cluster.
  <br><br>
  They must now display the newest version of Tanzu Application Platform.</td>
  <td>Run:
<pre>
tanzu package repository list -A
tanzu package installed list -A
</pre>
  </td>
</tr>

<tr>
  <td>Check the health of all kapp apps on the cluster</td>
  <td>Run: <pre>kapp list -A</pre></td>
</tr>

<tr>
  <td>Check the health of pods on the cluster</td>
  <td>Run: <pre>kubectl get pods -A --field-selector=status.phase!=Running,status.phase!=Succeeded</pre></td>
</tr>

<tr>
  <td>Check the health of exposed workload endpoints (knative routes and/or ingresses)</td>
  <td><em>n/a</em></td>
</tr>

<tr>
  <td>If applicable, confirm the status of:
  <ul>
  <li><code>APIDescriptors</code></li>
  <li>Service claims: <code>ClassClaim</code> and <code>ResourceClaim</code></li>
  </ul></td>
  <td><em>n/a</em></td>
</tr>

</tbody>
</table>
