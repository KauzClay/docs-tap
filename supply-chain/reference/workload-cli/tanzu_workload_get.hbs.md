# tanzu workload get

This topic gives you command-specific information for the Tanzu Supply Chain CLI.

## Synopsis

Get the details of the specified workload within a namespace.

Workload `get` configuration options include:

- Specify the namespace of the workload
- Specify the kind of the workload
- Output the workload formatted

```console
tanzu workload get <NAME> [flags]
```

## Examples

```console
tanzu workload get NAME
  tanzu workload get NAME --kind kindname
  tanzu workload get NAME --kind kindname --namespace default
```

## Options

```console
  -h, --help             help for get
  -k, --kind string      kind of the workload specification
  -n, --namespace name   kubernetes namespace (defaulted from kube config)
  -o, --output string    output the workload details formatted. Supported formats: "json", "yaml", "yml"
```

## Options inherited from parent commands

```console
      --kubeconfig file   kubeconfig file (default is $HOME/.kube/config)
      --no-color          deactivate color, bold, animations, and emoji output
```

## See also

- [tanzu workload](tanzu_workload.hbs.md) for creating, updating, viewing, and listing Tanzu
  Workloads