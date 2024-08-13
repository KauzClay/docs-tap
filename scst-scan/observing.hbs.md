# Observe Supply Chain Security Tools - Scan

This topic outlines observation and troubleshooting methods you can use with Supply Chain Security
Tools (SCST) - Scan components.

{{> 'partials/scst-scan/scan-1-0-deprecation' }}

## <a id="observability"></a> Observability

The scans run inside a Tekton `TaskRun` where the `TaskRun` creates a pod. Both the `TaskRun` and
pod are cleaned up after completion.

Before applying a new scan, you can set a watch on the `TaskRuns`, `Pods`, `SourceScans`, and
`Imagescans` to observe their progression by running:

```console
watch kubectl get sourcescans,imagescans,pods,taskruns,scantemplates,scanpolicies -n DEV-NAMESPACE
```

Where `DEV-NAMESPACE` is the developer namespace where the scanner is installed.