# Set up recurring scanning

This topic describes how to set up recurring scans by using Supply Chain Security Tools (SCST) -
Scan 2.0.

## <a id="overview"></a> Overview

Scan workloads with updated vulnerability databases frequently. Use the scanner of your choice to
scan images produced by your software supply chain, and any container images running in your Tanzu
Application Platform clusters.

Use SCST - Scan 2.0 to schedule container image scans with the following capabilities:

- Detect vulnerabilities without a supply chain run for the following container image sources:

  - Images produced by the supply chain using Tanzu Build Service or kaniko.
  - Images defined in a workload when using a prebuilt container image in your supply chain.
  - Images running in a Tanzu Application Platform cluster that were not produced by a software
    supply chain.

- Customize how far back you want to scan container images based on:

  - When the container image was created in the supply chain.
  - When the container image entered a running state on your Tanzu Application Platform cluster.

- Use one of the `ImageVulnerabilityScan` samples or create your own. For more information, see
  [ImageVulnerabilityScan samples](ivs-custom-samples.hbs.md#overview) or
  [Bring your own scanner using an ImageVulnerabilityScan](ivs-create-your-own.hbs.md).

- View detected vulnerabilities from your most recent image built for a workload in the Tanzu
  Developer Portal Supply Chain Choreographer plug-in.

- Use when either SCST - Scan 1.0 or SCST - Scan 2.0 is used to scan a recently built container in
  your supply chain.

## <a id="recurring-scanning-setup"></a> Set up recurring scanning

Unlike scans that occur as part of the software supply chain, scans invoked by recurring scanning
run as a centralized scan job in a single namespace. The scan job gathers images from all
namespaces. To set up recurring scanning, you must create a `RecurringImageVulnerabilityScan`. A
`RecurringImageVulnerabilityScan` defines:

- The interval in which to scan container images in crontab format.
- The number of images, created using the supply chain, to scan as determined by the selected number
  of days in the past.
- The number of images, started in your Tanzu Application Platform clusters, to scan as determined
  by the selected number of days in the past.
- The steps from the [IVS template](ivs-custom-samples.hbs.md) to use to scan your images, which
  define which scanner to use.
- The OCI-compliant registry to push the recurring scan results to.

### <a id="preqrequisites"></a> Before you begin

Before you define your `RecurringImageVulnerabilityScan` template, you must have:

- A repository created on an OCI-compliant registry that scan results are pushed to.
- A namespace in which to create the recurring scan template. This is where all the recurring scan
  jobs will run.
- A service account within the namespace that can push an OCI artifact to the results repository.
- Credentials for any registry from which the scanner must pull images to scan.

Recurring scanning uses the SCST - Scan 2.0 component, which is in the `Full` and `Build` profiles.

> **Note** Pay special attention to the service accounts and credentials that are needed. For a
> simplified setup of the namespace, VMware recommends that you use Namespace Provisioner to create
> a namespace. Namespace Provisioner automatically creates the service accounts and secrets needed
> for images created by supply chains. The examples in this topic use a namespace created by
> Namespace Provisioner. For more information, see
> [Namespace Provisioner](../namespace-provisioner/about.hbs.md).

### <a id="example-template"></a> Example `RecurringImageVulnerabilityScan` template

Use the Grype and Trivy samples in a namespace created by Namespace Provisioner. The Grype and Trivy
examples are a subset of this template. Add extra configurations from this template to the Grype and
Trivy samples for more advanced configurations.

The following sample template provides an explanation of the input variables for the
`RecurringImageVulnerabilityScan` CR:

```yaml
apiVersion: app-scanning.apps.tanzu.vmware.com/v1alpha1
kind: RecurringImageVulnerabilityScan
metadata:
  name: recurring-scan-template
spec:
  images:
    ranWithinDays: RAN-INTERVAL
    createdWithinDays: CREATED-INTERVAL
  retentionPolicy:
    maxFailedScans: FAILED-RETENTION
    maxSuccessfulScans: SUCCESSFUL-RETENTION
  schedule:
    cron: CRON-SCHEDULE
    startingDeadline: START-DEADLINE
  scan:
    activeKeychains:
    # Only enable keychains for registries you are using
    - name: acr  # Azure Container Registry
    - name: ecr  # Elastic Container Registry
    - name: gcr  # Google Container Registry
    - name: ghcr # Github Container Registry
    workspace:
      size: WORKSPACE-SIZE
    scanResults:
      location: RESULTS-REPOSITORY
    steps:
      STEPS-FROM-IVS-TEMPLATE
```

Where:

- `RAN-INTERVAL`: The number of earlier days of images from pods that have started on the Tanzu
  Application Platform clusters to scan.
- `CREATED-INTERVAL` The number of earlier days of images from supply chains to scan.
- `FAILED-RETENTION`: The number of failed recurring scan executions to keep in Kubernetes.
- `CRON-SCHEDULE`: The schedule in which to invoke recurring scans in crontab format. For example,
  to execute a scan daily at 3:00 AM, the value is `0 3 * * *`
- `START-DEADLINE`: The period of time beyond the scheduled start time when scans can be started if
  they did not start on time. If this period elapses, the scheduled scan is skipped.
- `SUCCESSFUL-RETENTION`: The number of successful recurring scan executions to keep in Kubernetes.
- `WORKSPACE-SIZE`: The size of the workspace used when scanning images. This is created as a
  Kubernetes PVC. This depends mostly on the size of the vulnerability database, the number of
  images to scan, and the output of the vulnerability scanner. `10Gi` is the recommended
  starting point.
- `RESULTS-REPOSITORY`: The registry URL where results are uploaded. For example,
  `my.registry/scan-results`.
- `STEPS-FROM-IVS-TEMPLATE`: The steps to scan the list of the container images. For commonly used
  samples, see [ImageVulnerabilityScan samples](ivs-custom-samples.hbs.md#overview). To pass
  `spec.image` and `scanResults.location` to `args`, you can use `{image}` and `{output}`
  respectively. These interpolation variables differ from the ones currently used in the
  `ImageVulnerabilityScan` (`$(params.image)` and `$(params.scan-results-path)`). The scan step must
  the last step in the `steps` property.

> **Note** Pay special attention to the scanning runtime, the scan frequency, and the configured
> deadline. Setting `RAN-INTERVAL` and `CREATED-INTERVAL` to a large number of days can increase the
> number of images scanned, and in turn, the scanning runtime. If the scan runtime is greater than
> the `CRON-SCHEDULE` frequency, the scans start to queue up. VMware recommends running the scan
> once every 24 hours at a time when your cluster load is low.

### <a id="grype-rivs-template"></a> Grype `RecurringImageVulnerabilityScan` template

The SCST - Scan 1.0 default scanner is Grype, and this template works with the SCST - Scan 1.0
default configuration.

> **Note** You must match the scanner and version to the scanner and version used in the software
> supply chain. Using different types of scanners between build time and recurring scans is not
> supported and causes the Security Analysis plug-in in Tanzu Developer Portal to double count
> vulnerabilities.

To apply this configuration, save it to a file and apply it to the namespace created for recurring
scans by running:

```console
kubectl apply -f grype-recurring-scan.yaml  --namespace NAMESPACE
```

This sample configuration downloads the latest Grype vulnerability database and then scans with the
stored database, which prevents multiple database updates while running concurrent scans:

```yaml
apiVersion: app-scanning.apps.tanzu.vmware.com/v1alpha1
kind: RecurringImageVulnerabilityScan
metadata:
  name: grype-recurring-scan
spec:
  images:
    createdWithinDays: 180
    ranWithinDays: 0
  retentionPolicy:
    maxFailedScans: 1
    maxSuccessfulScans: 1
  schedule:
    cron: 0 3 * * *
    startingDeadline: 60m
  scan:
    workspace:
      size: 10Gi
    scanResults:
      location: RESULTS-REPOSITORY
    steps:
    - name: update-db
      image: anchore/grype:latest
      command: [/grype]
      args:
      - db
      - update
      env:
      - name: GRYPE_DB_CACHE_DIR
        value: /workspace/grype-cache
    - name: grype-scan
      image: anchore/grype:latest
      command: [/grype]
      args:
      - "{image}"
      - "-o"
      - "cyclonedx-json"
      - "--file"
      - "{output}"
      env:
      - name: GRYPE_DB_AUTO_UPDATE
        value: "false"
      - name: GRYPE_CHECK_FOR_APP_UPDATE
        value: "false"
      - name: GRYPE_DB_CACHE_DIR
        value: /workspace/grype-cache
```

The placeholder `{image}` is replaced by the container images that are being scanned.
This is equivalent to the `$(params.image)` placeholder used in the `ImageVulnerabilityScan`.

The placeholder `{output}` is replaced by an auto-generated path to store the results for the
scanned image. This is equivalent to the `$(params.scan-results-path)` placeholder used in the
`ImageVulnerabilityScan`.

### <a id="trivy-rivs-template"></a> Trivy `RecurringImageVulnerabilityScan` template

The SCST - Scan 2.0 default scanner is Trivy, and this template works with the Scan 2.0 default
configuration.

> **Note** You must match the scanner and version to the scanner and version used in the software
> supply chain. Using different types of scanners between build time and recurring scans is not
> supported, and causes the Security Analysis plug-in in Tanzu Developer Portal to double count
> vulnerabilities.

To apply this configuration, save it to a file and apply it to the namespace created for recurring
scans by running:

```console
kubectl apply -f trivy-recurring-scan.yaml --namespace NAMESPACE
```

This sample configuration downloads the latest Trivy vulnerability and Java databases and then scans
with the stored databases, which prevents multiple database updates while running concurrent scans:

```yaml
apiVersion: app-scanning.apps.tanzu.vmware.com/v1alpha1
kind: RecurringImageVulnerabilityScan
metadata:
  name: trivy-recurring-scan
spec:
  images:
    createdWithinDays: 180
    ranWithinDays: 0
  retentionPolicy:
    maxFailedScans: 1
    maxSuccessfulScans: 1
  schedule:
    cron: 0 3 * * *
    startingDeadline: 60m
  scan:
    workspace:
      size: 10Gi
    scanResults:
      location: RESULTS-REPOSITORY
    steps:
    - name: update-db
      image: aquasec/trivy:0.48.3
      command: [/usr/local/bin/trivy]
      args:
      - image
      - --download-db-only
      - --no-progress
      - --cache-dir=/workspace/trivy-cache
    - name: update-java-db
      image: aquasec/trivy:0.48.3
      command: [/usr/local/bin/trivy]
      args:
      - image
      - --download-java-db-only
      - --no-progress
      - --cache-dir=/workspace/trivy-cache
    - name: trivy-generate-report
      image: aquasec/trivy:0.48.3
      command: [/usr/local/bin/trivy]
      args:
      - image
      - '{image}'
      - --exit-code=0
      - --no-progress
      - --scanners=vuln
      - --format=cyclonedx
      - --output={output}
      - --cache-dir=/workspace/trivy-cache
      - --skip-db-update
      - --skip-java-db-update
```

The placeholder `{image}` is replaced by the container images being scanned.
This is equivalent to the `$(params.image)` placeholder used in the `ImageVulnerabilityScan`.

The placeholder `{output}` is replaced by an auto-generated path to store the results for the
scanned image. This is equivalent to the `$(params.scan-results-path)` placeholder used in the
`ImageVulnerabilityScan`.

> **Note** Do not enclose the `{output}` interpolation value in quotation marks for a Trivy scan.
