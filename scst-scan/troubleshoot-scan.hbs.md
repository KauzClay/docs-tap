# Troubleshoot Supply Chain Security Tools - Scan

This topic describes troubleshooting methods you can use with Supply Chain Security Tools (SCST) -
Scan.

## <a id="debugging-commands"></a> Debugging commands

Run these commands to get more logs and details about the errors around scanning. The `TaskRuns` and
pods persist for a predefined amount of seconds before getting deleted.
(`deleteScanJobsSecondsAfterFinished` is the tap pkg variable that defines this)

### <a id="debug-tekton-taskrun"></a> Debug Tekton `TaskRun`

To retrieve `TaskRun` events run:

```console
kubectl describe taskrun TASKRUN-NAME -n DEV-NAMESPACE
```

Where:

- `TASKRUN-NAME` is the name of the `TaskRun`.
- `DEV-NAMESPACE` is the name of the developer namespace you want to use.

### <a id="debugging-scan-pods"></a> Debug Scan pods

Run the following to get error logs from a pod when scan pods are in a failing state:

```console
kubectl logs scan-pod-name -n DEV-NAMESPACE
```

Where `DEV-NAMESPACE` is the name of the developer namespace you want to use.

For more information, about debugging Kubernetes pods see
[here](https://jamesdefabia.github.io/docs/user-guide/kubectl/kubectl_logs/)

The following is an example of a successful scan run output:

```yaml
scan:
  cveCount:
    critical: 20
    high: 120
    medium: 114
    low: 9
    unknown: 0
  scanner:
    name: Grype
    vendor: Anchore
    version: v0.37.0
  reports:
  - /workspace/scan.xml
eval:
  violations:
  - CVE node-fetch GHSA-w7rc-rwvf-8q5r Low
store:
  locations:
  - https://metadata-store-app.metadata-store.svc.cluster.local:8443/api/sources?repo=hound&sha=5805c6502976c10f5529e7f7aeb0af0c370c0354&org=houndci
```

A scan run that has an error means that one of the init containers: `scan-plugin`,
`metadata-store-plugin`, `compliance-plugin`, `summary`, or any other additional containers had a
failure.

To inspect for a specific `init` container in a pod run:

```console
kubectl logs scan-pod-name -n DEV-NAMESPACE -c init-container-name
```

Where `DEV-NAMESPACE` is the name of the developer namespace you want to use.

For debug init container tips, see
[Debug Init Containers](https://kubernetes.io/docs/tasks/debug/debug-application/debug-init-containers/)
in the Kubernetes documentation.

### <a id="debug-source-image-scan"></a> Debug `SourceScan` and `ImageScan`

To retrieve status conditions of an `SourceScan` and an `ImageScan`, run:

```console
kubectl describe sourcescan SOURCE-SCAN -n DEV-NAMESPACE
```

Where:

- `DEV-NAMESPACE` is the name of the developer namespace you want to use.
- `SOURCE-SCAN` is the name of the `SourceScan` you want to use.

```console
kubectl describe imagescan IMAGE-SCAN -n DEV-NAMESPACE
```

Where:

- `DEV-NAMESPACE` is the name of the developer namespace you want to use.
- `IMAGE-SCAN` is the name of the `ImageScan` you want to use.

Under `Status.Conditions`, for a condition look at the `Reason`, `Type`, and `Message` values that
use the keyword `Error` to investigate issues.

### <a id="debug-scanning-in-sc"></a> Debug Scanning within a `SupplyChain`

For Tanzu workload commands for tailing build and runtime logs, and getting workload status and
details, see [Troubleshoot Apps CLI](../cli-plugins/apps/troubleshooting/troubleshooting.hbs.md).

### <a id="view-scm-logs"></a> View the `scan-controller` manager logs

To retrieve `scan-controller` manager logs:

```console
kubectl -n scan-link-system logs -f deployment/scan-link-controller-manager -c manager
```

## <a id="restarting-deployment"></a> Restarting Deployment

If you encounter an issue with the scan-link controller not starting, run the following to restart
the deployment to see if it is reproducible or flaking when starting:

```console
kubectl rollout restart deployment scan-link-controller-manager -n scan-link-system
```

## <a id="scanner-mds-config"></a> Troubleshoot the scanner to Metadata Store configuration

If you encounter this issue:

```console
✖  Error: Post "https://metadata-store.tap.tanzu.example.com/api/sourceReport?": \
tls: failed to verify certificate: x509: certificate signed by unknown authority
```

ensure that the `caSecret` from the scanner `DEV-NAMESPACE` matches the `caSecret` from the
`METADATASTORE-NAMESPACE` namespace:

1. If you provided an ingress connection to the store, delete it. In a single cluster, the
   connection between the scanning pod and the Metadata Store happens inside the cluster and does
   not pass through ingress. This is automatically configured. You do not need to provide an ingress
   connection to the store.

1. Get the `caSecret.name`. If you are using a multicluster setup, retrieve
   `grype.metadataStore.caSecret.name` from the Grype configuration as in this example:

    ```yaml
    grype:
      metadataStore:
        caSecret:
          name: store-ca-cert
          importFromNamespace: metadata-store-secrets
    ```

   > **Note** `caSecret.name` is set to `store-ca-cert`.
   > For more information, see
   > [Set up multicluster Supply Chain Security Tools (SCST) - Scan 1.0](../scst-scan/multicluster-setup-scan-v1.hbs.md).

   If you are using a single-cluster setup, the default value for
   `grype.metadataStore.caSecret.name` is `app-tls-cert`. For more information, see
   [Install Supply Chain Security Tools - Scan](./install-scst-scan.hbs.md#configure-properties).

1. Verify that the `CA-SECRET` secret exists in the `DEV-NAMESPACE` by running:

   ```console
   kubectl get secret CA-SECRET -n DEV-NAMESPACE
   ```

1. If the secret `CA-SECRET` doesn't exist in your `DEV-NAMESPACE`, verify that the `CA-SECRET`
   exists in the `METADATASTORE-NAMESPACE` namespace by running:

   ```console
   kubectl get secret CA-SECRET -n METADATASTORE-NAMESPACE
   ```

   Where `METADATASTORE-NAMESPACE` is the namespace that contains the secret `CA-SECRET`.

   If you are using a single cluster, it is configured by using the `metadata-store` namespace. If
   multicluster, it is configured by using `metadata-store-secrets`.

   If `CA-SECRET` does not exist in the Metadata Store namespace, see
   [Custom certificate configuration](../scst-store/custom-cert.hbs.md) to configure the
   certificate.

1. Verify that the `secretexport` and `secretimport` exist and are reconciling successfully by
   running:

   ```console
   kubectl get secretexports.secretgen.carvel.dev -n `METADATASTORE-NAMESPACE`
   kubectl get secretimports.secretgen.carvel.dev -n `DEV-NAMESPACE`
   ```

   SCST - Store creates the single-cluster `secretexport` by default. For more information, see
   [Deployment details and configuration](../scst-store/deployment-details.hbs.md#exporting-certificates).
   For information about creating the multicluster `secretexport`, see
   [Set up multicluster SCST - Scan 1.0](../scst-scan/multicluster-setup-scan-v1.hbs.md#export-multicluster).

1. Verify that the `ca.crt` field in both secrets from `METADATASTORE-NAMESPACE` and `DEV-NAMESPACE`
   match, or that the `ca.crt` field of the secret in the `METADATASTORE-NAMESPACE` includes the
   `ca.crt` field of the `DEV-NAMESPACE` secret.

   To do so, base64-decode both secrets and verify that there is a match by running:

   ```console
   kubectl get secret CA-SECRET -n DEV-NAMESPACE -o json | jq -r '.data."ca.crt"' | base64 -d
   kubectl get secret CA-SECRET -n METADATASTORE-NAMESPACE -o json | jq -r '.data."ca.crt"' | base64 -d
   ```

  The certificates in the `METADATASTORE-NAMESPACE` and `DEV-NAMESPACE` must have a match for the
  scanner to connect to `metadata-store`.

## <a id="troubleshooting-issues"></a> Troubleshooting issues

See the following sections for troubleshooting specific known issues.

### <a id="source-scan-missing"></a> Source scan missing in supply chain

The source scan step is opt-in in Tanzu Application Platform v1.6 and later to better support
languages that resolve dependencies at build time. For more information and how to opt in to source
scanning in the out-of-the-box test and scan supply chain, see
[Scan Types for Supply Chain Security Tools - Scan](scan-types.hbs.md#source-scan).

### <a id="troubleshoot-grype-airgap"></a> Troubleshoot Grype in air-gapped environments

For information about issues with Grype in air-gapped environments, see
[Use vulnerability scanning in offline and air-gapped environments](offline-airgap.hbs.md).

### <a id="miss-src-ps"></a> Missing target SSH secret

Scanning source code from a private source repository requires an SSH secret in the namespace and
referenced as `grype.targetSourceSshSecret` in `tap-values.yaml`. For more information, see
[Installing the Tanzu Application Platform Package and Profiles](../install-online/profile.hbs.md).

If a private source scan is triggered and the secret cannot be found, the scan pod includes a
`FailedMount` warning in Events with the message:

```console
MountVolume.SetUp failed for volume "ssh-secret" : secret "secret-ssh-auth" not found
```

Where `secret-ssh-auth` is the value specified in `grype.targetSourceSshSecret`

### <a id="miss-img-ps"></a> Missing target image pull secret

Scanning an image from a private registry requires an image pull secret to exist in the Scan CRs
namespace and to be referenced as `grype.targetImagePullSecret` in `tap-values.yaml`. For more
information, see
[Installing the Tanzu Application Platform Package and Profiles](../install-online/profile.hbs.md).

If a private image scan is triggered and the secret is not configured, the scan `TaskRun` pod's
`step-scan-plugin` container fails with the following error:

```console
Error: GET https://tanzu.packages.broadcom.com/v2/vse-dev/spring-petclinic/manifests/sha256:128e38c1d3f10401a595c253743bee343967c81e8f22b94e30b2ab8292b3973f: \
UNAUTHORIZED: unauthorized to access repository: vse-dev/spring-petclinic, action: pull: unauthorized to access repository: vse-dev/spring-petclinic, action: pull
```

### <a id="deactivate-scst-store"></a> Deactivate SCST - Store

SCST - Store is required to install SCST - Scan:

1. If you install without SCST - Store, edit the configuration to deactivate the Store as follows:

    ```yaml
    ---
    metadataStore:
      url: ""
    ```

1. Install the package with the edited configuration by running:

   ```console
   tanzu package install scan-controller \
     --package scanning.apps.tanzu.vmware.com \
     --version VERSION \
     --namespace tap-install \
     --values-file tap-values.yaml
   ```

### <a id="incompatible-ssv"></a> Resolve incompatible Syft Schema Version

You might encounter the following error:

```console
The provided SBOM has a Syft Schema Version which doesn't match the version that is supported by Grype...
```

This means that the Syft Schema Version from the provided SBOM does not match the version supported
by the installed `grype-scanner`. There are two different methods to resolve this incompatibility
issue:

- (Recommended) Install a version of [Tanzu Build Service](../tanzu-build-service/tbs-about.hbs.md)
  that provides an SBOM with a compatible Syft Schema Version.

- Deactivate `failOnSchemaErrors` in `grype-values.yaml` as follows:

    ```yaml
    syft:
      failOnSchemaErrors: false
    ```

  For more information, see [Install Supply Chain Security Tools - Scan](install-scst-scan.hbs.md).
  Although this change bypasses the check on Syft Schema Version, it does not resolve the
  incompatibility issue and produces a partial scanning result.

### <a id="incompatible-scan-policy"></a> Resolve incompatible scan policy

If your scan policy appears to not be enforced, it might be because the Rego file defined in the
scan policy is incompatible with the scanner that is being used. For example, the Grype Scanner
outputs in the CycloneDX XML format while the Snyk Scanner outputs SPDX JSON.

For an example of a `ScanPolicy` formatted for SPDX JSON, see
[Sample ScanPolicy for Snyk in SPDX JSON format](install-snyk-integration.hbs.md#snyk-scan-policy)

### <a id="ca-not-found-in-secret"></a> Could not find CA in secret

If you encounter the following issue, it might be because of not exporting `app-tls-cert` to the
correct namespace:

```console
{"level":"error","ts":"2022-06-08T15:20:48.43237873Z","logger":"setup","msg":"Could not find CA in Secret","err":"unable to set up connection to Supply Chain Security Tools - Store"}
```

Configure `ns_for_export_app_cert` in your `tap-values.yaml` file as follows:

```yaml
metadata_store:
  ns_for_export_app_cert: "DEV-NAMESPACE"
```

Where `DEV-NAMESPACE` is the name of the developer namespace you want to use.

If there are multiple developer namespaces, use `ns_for_export_app_cert: "*"`.

### <a id="reporting-wrong-blob-url"></a> Blob Source Scan is reporting the wrong source URL

A Source Scan for a blob artifact can cause reporting the wrong URL for the resource in
`status.artifact` and `status.compliantArtifact`, passing the remote SSH URL instead of the cluster
local `fluxcd` URL. One symptom of this issue is `image-builder` failing with a
`ssh:// is an unsupported protocol` error message.

Verify that this is the problem by running: `kubectl describe` in the affected resource and

```console
kubectl describe sourcescan SOURCE-SCAN-NAME -n DEV-NAMESPACE
```

Where:

- `SOURCE-SCAN-NAME` is the name of the source scan you want to configure.
- `DEV-NAMESPACE` is the name of the developer namespace you want to use.

Compare `spec.blob.url` value and `status.artifact.blob.url` to see if they are different URLs. For
example:

```yaml
...
spec:
  blob:
    ...
    url: http://source-controller.flux-system.svc.cluster.local./gitrepository/sample/repo/8d4cea98b0fa9e0112d58414099d0229f190f7f1.tar.gz
    ...
status:
  artifact:
    blob:
      ...
      url: ssh://git@github.com:sample/repo.git
  compliantArtifact:
    blob:
      ...
      url: ssh://git@github.com:sample/repo.git
```

This problem happens in SCST - Scan v1.2.0 when you use a Grype Scanner `ScanTemplates` resource
earlier than v1.2.0 because this is a deprecated path. To fix this problem, upgrade your Grype
Scanner deployment to v1.2.0 or later. For instructions, see
[Upgrade to Version v1.2.0](upgrading.hbs.md#upgrade-to-1-2-0)

### <a id="supply-chain-stops"></a> Resolving failing scans that block a Supply Chain

If the Supply Chain is not progressing because of CVEs found in either the `SourceScan` or `ImageScan`,
see the CVE triage workflow in
[Triaging and Remediating CVEs](triaging-and-remediating-cves.hbs.md).

### <a id="tdp-miss-policy"></a> Policy not defined in Tanzu Developer Portal

If you encounter `No policy has been defined`, it might be because Tanzu Developer Portal is unable
to view the Scan Policy resource. For example, the `scanPolicy` in the scan matches the name of the
Scan Policy.

Confirm that the Scan Policy associated with a `SourceScan` or `ImageScan` exists by running:

```console
kubectl describe sourcescan NAME -n DEV-NAMESPACE
kubectl describe imagescan NAME -n DEV-NAMESPACE
kubectl get scanpolicy NAME -n DEV-NAMESPACE
```

Where `DEV-NAMESPACE` is the name of the developer namespace you want to use.

Add the `app.kubernetes.io/part-of` label to the Scan Policy. For more information, see
[Enable Tanzu Developer Portal to view the ScanPolicy resource](policies.hbs.md#gui-view-scan-policy).

### Lookup error when connecting to SCST - Store

If your scan pod is failing, you might see the following connection error in the logs:

```console
dial tcp: lookup metadata-store-app.metadata-store.svc.cluster.local on 10.100.0.10:53: no such host
```

A connection error while attempting to connect to the local cluster URL causes this error. If this
is a multicluster deployment, set the `grype.metadataStore.url` property in your Build profile
`values.yaml` file. You must set the ingress domain of SCST - Store, which is deployed in the View
cluster. For information about this configuration, see
[Configure Grype in the Build profile values file](../scst-scan/multicluster-setup-scan-v1.hbs.md#grype-mds-config).

### `SourceScan` error with SCST - Store endpoint without a prefix

If your `SourceScan` resource is failing, the status might show this error:

```console
Error: endpoint require 'http://' or 'https://' prefix
```

This is because the `grype.metadataStore.url` value in the Tanzu Application Platform profile
`values.yaml` file was not configured with the correct prefix. Verify that the URL starts with
`http://` or `https://`.

### <a id="deprecated-templates"></a> Deprecated pre-v1.2 templates

If the scan phase is in `Error` and the status condition message is:

```console
Summary logs could not be retrieved: . error opening stream pod logs reader: container summary is \
not valid for pod scan-grypeimagescan-sample-public-zmj2g-hqv5g
```

this error might be a consequence of using Grype Scanner `ScanTemplates` shipped with SCST - Scan v1.1
or earlier. These `ScanTemplates` are deprecated and are not supported in Tanzu Application Platform
v1.4.0 and later.

There are two options to resolve this issue:

- Option 1: Upgrade to the latest Grype Scanner version. This automatically replaces the old
  `ScanTemplates` with the upgraded `ScanTemplates`.

- Option 2: Create a `ScanTemplate` by following the steps in
  [Create a scan template](create-scan-template.hbs.md).

### <a id="inc-cnfg-self-signed-cert"></a> Incorrectly configured self-signed certificate

The following error in the pod logs indicate that the self-signed certificate might be incorrectly
configured:

```console
x509: certificate signed by unknown authority
```

To resolve this issue, ensure that `shared.ca_cert_data` contains the required certificate. For an
example of setting up the shared self-signed certificate, see
[Build profile](../multicluster/reference/tap-values-build-sample.hbs.md).

For information about `shared.ca_cert_data`, see
[View possible configuration settings for your package](../install-online/view-package-config.hbs.md).

### <a id="unable-pull-scanner-imgs"></a> Unable to pull scan controller and scanner images from a specified registry

The `docker` field and related sub-fields by SCST - Scan Controller, Grype Scanner, or Snyk Scanner
are deprecated in Tanzu Application Platform v1.4.0 and removed in Tanzu Application Platform
v{{ vars.tap_version }}. Previously these fields might be used to populate the `registry-credentials`
secret. You might encounter the following error during installation:

```console
UNAUTHORIZED: unauthorized to access repository
```

The recommended migration path for users setting up their namespaces manually is to add registry
credentials to both the developer namespace and the `scan-link-system` namespace, using these
[instructions](../install-online/set-up-namespaces.hbs.md).

> **Important** This step does not apply to users who used `--export-to-all-namespaces` when setting
> up the Tanzu Application Platform package repository.

### <a id="grype-db-not-available"></a> Grype database not available

Before running a scan, the Grype scanner downloads a copy of its database. If the database fails to
download, the following log entry might appear.

```console
Vulnerability DB [no update available] New version of grype is available: 0.50.2 [0000] WARN unable \
to check for vulnerability database update 1 error occurred: * failed to load vulnerability db: \
vulnerability database is corrupt (run db update to correct): database metadata not found: \
~/Library/Caches/grype/db/3
```

To resolve this issue, ensure that Grype has access to its vulnerability database:

- If you set up a mirror of the vulnerability database in
  [Use vulnerability scanning in offline and air-gapped environments](offline-airgap.hbs.md),
  verify that it is populated and reachable.
- If you did not set up a mirror, Grype manages its database behind the scenes. Verify that the
  cluster has access to `https://anchore.com/`.

This issue is unrelated to SCST – Store.

### <a id="scanner-pod-restarts"></a> A Scanner Pod restarts once in SCST - Scan v1.5.0 or later

For SCST - Scan `v1.5.0` or later, you see scanner pods restart:

```console
Pods
   NAME                                  READY   STATUS      RESTARTS   AGE
   my-scan-45smk-pod                     0/9     Completed   1          14m
```

One restart in scanner pods is expected with successful scans. To support Tanzu Service Mesh (TSM)
integration, jobs were replaced with `TaskRuns`. This restart is an artifact of how Tekton cleans up
sidecar containers by patching the container specifications.

### <a id="recon-fail-during-upgrade"></a> Reconciliation of SCST - Scan fails when upgrading to v{{ vars.url_version }}

When upgrading the SCST - Scan from a previous version to v{{ vars.url_version }}, you might see a
reconciliation failure similar to:

```console
NAME                                PACKAGE-NAME                                         PACKAGE-VERSION                STATUS
scanning                            scanning.apps.tanzu.vmware.com                       {{ vars.tap_version }}-build.36081392+c072d305  Reconcile failed
```

If you get the package by running

```console
tanzu package installed get scanning -n tap-install
```

you might see an error similar to:

```console
STATUS:                  Reconcile failed
CONDITIONS:              - type: ReconcileFailed
  status: "True"
  reason: ""
  message: Error (see .status.usefulErrorMessage for details)
USEFUL-ERROR-MESSAGE:    ytt: Error: Overlaying data values (in following order: additional data values):
One or more data values were invalid
====================================Given data value is not declared in schema
values.yaml:
    |
  2 |   url: ""
    |    = found: url
    = expected: a map item with the key named "exports" (from .ytt/data.yaml:52)
```

This is because the field `scanning.metadataStore.url` is removed. If this field is in the
`tap-values.yaml` file provided for the upgrade, the reconciliation fails. To resolve this problem,
remove the field from the values file and run the upgrade command again.

### <a id="scanning-restricted-kpss"></a> Scanning in a cluster with restricted Kubernetes Pod Security Standards

As part of compliance with the restricted profile Kubernetes Pod Security Standards, you must set
the `securityContext` of containers and `initContainers`. This applies to the `prepare` `initContainers`
created by Tekton. When a pod does not meet pod Security Standards, it is not created and
vulnerability scanning cannot proceed. For more information, see the
[Kubernetes documentation](https://kubernetes.io/docs/concepts/security/pod-security-standards/).

You might see an error message similar to the following when describing the `TaskRun`:

```console
"scan-source-scan-with-passing-policy-zx46t-pod" is forbidden: violates PodSecurity "restricted:latest": \
allowPrivilegeEscalation != false (container "prepare" must set securityContext.allowPrivilegeEscalation=false), \
unrestricted capabilities (container "prepare" must set securityContext.capabilities.drop=["ALL"]), \
seccompProfile (pod or container "prepare" must set securityContext.seccompProfile.type to \
"RuntimeDefault" or "Localhost"). Maybe invalid TaskSpec. ScanPodError PodNotFound: no pod found
```

1. Edit your Tekton Pipelines package configuration in your `tap-values.yaml` file with the
   following changes:

    ```yaml
    tekton_pipelines:
        feature_flags:
            set_security_context: "true"
    ```

   By setting the `securityContext` you resolve the `prepare` `initContainer` violation.

2. Update your Tanzu Application Platform installation by running:

   ```console
   tanzu package installed update tap -p tap.tanzu.vmware.com -v TAP-VERSION  --values-file \
   tap-values.yaml -n tap-install
   ```

   Where `TAP-VERSION` is the version of Tanzu Application Platform installed.

3. Run the scan again.
