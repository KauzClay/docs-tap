# tanzu workload logs

Watch workload related logs

## Synopsis

Stream logs for a workload until canceled. To cancel, press Ctrl+C in
the shell or stop the process. As new workload pods are started, the logs
are displayed. To show historical logs use --since.

```console
tanzu workload logs <NAME> [flags]
```

## Examples

```console
tanzu workload logs NAME
  tanzu workload logs NAME --since 1h
  tanzu workload logs NAME --since 1h --namespace default
  tanzu workload logs NAME --since 1h --namespace default --run runname
```

## Options

```console
  -h, --help             help for logs
  -k, --kind string      kind of the workload specification
  -n, --namespace name   kubernetes namespace (defaulted from kube config)
      --run name         workload run name
      --since duration   time duration to start reading logs from (default 1h0m0s)
  -t, --timestamp        print timestamp for each log line
```

## Options inherited from parent commands

```console
      --kubeconfig file   kubeconfig file (default is $HOME/.kube/config)
      --no-color          deactivate color, bold, animations, and emoji output
```

## SEE ALSO

* [tanzu workload](tanzu_workload.hbs.md)	 - create, update, view and list Tanzu Workloads.

