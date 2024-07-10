# Exporting Telemetry Data (Experimental)

This section explains how to export the Open Telemetry data, and what information is currently exported.

NOTE: This feature is experimental, and the exported information may change over time.

Tanzu Supply Chain uses [OpenTelemetry](https://opentelemetry.io/) to export telemetry data. You can refer to
their website for a list of vendors and observability tools that support OpenTelemetry.

## Configuring Supply Chain to export Telemetry Data

To export telemetry data to an OpenTelemetry collector, update the
`tanzu_supply_chain` section of `tap-values.yaml` to include the collector's address. For example:

```yaml
tanzu_supply_chain:
  ...
  open_telemetry:
    endpoint: "https://tempo.monitoring:4318"
    insecure: "false"
```

## What Data is Exported

The supply chain exports the following telemetry information: `WorkloadRun` and `Stage` information.

`WorkloadRun` information includes when a particular instance of a workload was triggered and its duration.

`Stage` information includes when it was initiated, its duration, and which `WorkloadRun` it is associated with.

When the telemetry data is rendered in a visualizer, it will look similar to the following:

![otel-example.png](images/otel-example.png)

`WorkloadRun` instance `app1-run-bkmdx` took ~58s to complete and involved four stages. Each `Stage` displays when 
they are triggered and their execution duration.

## Known Limitation

Currently, `WorkloadRun` and `Stage` telemetry information are only published when they finish. Therefore, `Stage` telemetry 
data can appear before their parent `WorkloadRun` telemetry data during a run's execution. Once the run completes, all 
telemetry data associated with that run, including its `Stages`, will be published.
