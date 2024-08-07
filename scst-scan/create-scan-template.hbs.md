# Create a ScanTemplate with Supply Chain Security Tools - Scan

This topic tells you how to create a `ScanTemplate` with Supply Chain Security Tools (SCST) - Scan.

{{> 'partials/scst-scan/scan-1-0-deprecation' }}

## Overview

The `ScanTemplate` custom resource (CR) defines how the scan Pod fulfills the task of vulnerability
scanning. There are default `ScanTemplate`s, provided as standard, that use the Tanzu Application
Platform default scanner Anchore Grype. One or more `initContainer`s run to complete the scan and
must save results to a shared `volume`.

After the `initContainer`s finishes, a single container on the scan Pod called `summary` combines
the result of the `initContainer`s so that the `Scan CR` status is updated.

A customized `ScanTemplate` is created by editing or replacing `initContainer` definitions and
reusing the `summary` container from the `grype` package. A container can read the `out.yaml` from
an earlier step to locate relevant inputs.

## <a id="output-model"></a> Output model

Each `initContainer` can create a subdirectory in `/workspace` to use as a scratch space. Before
terminating, the container must create an `out.yaml` file in the subdirectory containing the relevant
subset of fields from the output model:

```yaml
fetch:
  git:
    url:
    revision:
    path:
  blob:
    url:
    revision:
    path:
  image:
    url:
    revision:
    path:
sbom:
    packageCount:
    reports: []
scan:
  cveCount:
    critical:
    high:
    medium:
    low:
    unknown:
  scanner:
    name:
    vendor:
    version:
    db:
      version:
  reports: []
eval:
  violations: []
store:
  locations: []
```

The `scan` portion of the earlier output is required and, if missing, the scan controller fails to
properly update the final status of the `Scan CR`. Other portions of the output, including those of
`store` and `policy evaluation`, are optional and can be omitted if not applicable in a custom
supply-chain setup.

## <a id="template-structure"></a> `ScanTemplate` structure

This is an example `ScanTemplate` structure:

```yaml
apiVersion: scanning.apps.tanzu.vmware.com/v1beta1
kind: ScanTemplate
spec:
    template: # a core/v1 PodSpec
      # Here are list volumes mounted for writing to or
      # reading from during different stages of the scan
      volumes:
        # required the results of different scan stages
        # should be saved in files digestible by the scan
        # controller in this volume
        - name: workspace
        emptyDir: { }
      # different steps required for a scanning can be staged
      # in sequential stages through initContainers.
      initContainers:
      # Summary container will take results of initContainers
      # and will let Controller to update Scan CR status.
      containers:
        - name: summary
```

> **Note** You cannot name a container `sleep` because there is already a container named `sleep`
> that comes from the `scan-link` controller.

## <a id="sample-output"></a> Sample Outputs

This is an example for a typical Git clone (source scan fetch stage) that is saved in
`/workspace/git-clone/out.yaml`:

```yaml
fetch:
  git:
    url: github.com/my/repo
    revision: aee9f8
    path: /workspace/git-clone/cloned-repository
```

This is an example of a typical scan stage, which is saved in `/workspace/grype-scan/out.yaml`:

```yaml
scan:
  cveCount:
    critical: 0
    high: 1
    medium: 3
    low: 25
    unknown: 0
  scanner:
    name: grype
    vendor: Anchore
    version: 0.33.0
    db:
      version: 2022-04-13
  reports:
  - /workspace/grype-scan/repo.cyclonedx.xml
  - /workspace/grype-scan/app.cyclonedx.xml
  - /workspace/grype-scan/base.cyclonedx.xml
```

This is an example of a typical evaluation stage, which is saved in
`/workspace/policy-eval/out.yaml`:

```yaml
eval:
  violations:
    - banned package log4j
    - critical CVE 2022-01-01-3333
    - number of critical CVEs over threshold
```

This is an example of a typical upload-to-store stage, which is saved in
`/workspace/upload-to-store/out.yaml`:

```yaml
store:
  locations:
    - http://metadata-store.cluster.local:8080/reports/3
```
