# tanzu workload run get

Get workload run

## Synopsis

Get the details of the specified workload run within a namespace.

workload run get configuration options include:
- Specify the namespace of the workload run
- Output the workload run formatted
- Show more details of the workload run

```console
tanzu workload run get <NAME> [flags]
```

## Examples

```console
tanzu workload run get NAME
  tanzu workload run get NAME --namespace namespace
  tanzu workload run get NAME --namespace namespace --show-details
```

## Options

```console
  -h, --help             help for get
  -n, --namespace name   kubernetes namespace (defaulted from kube config)
  -o, --output string    output the workload run details formatted. Supported formats: "json", "yaml", "yml"
      --show-details     show more details of the workload run
```

## Options inherited from parent commands

```console
      --kubeconfig file   kubeconfig file (default is $HOME/.kube/config)
      --no-color          deactivate color, bold, animations, and emoji output
```

## SEE ALSO

* [tanzu workload run](tanzu_workload_run.hbs.md)	 - View workload runs

