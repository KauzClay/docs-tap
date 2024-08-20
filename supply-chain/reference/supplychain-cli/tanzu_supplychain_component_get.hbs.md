# tanzu supplychain component get

This topic gives you command-specific information for the Tanzu Supply Chain CLI.

## Synopsis

Get the details of the specified component within a namespace.

`component get` configuration options include:

- Specify the namespace of the component (optional)
- Output the component formatted
- Show more details of the component

```console
tanzu supplychain component get <NAME> [flags]
```

## Examples

```console
tanzu supplychain component get NAME
  tanzu supplychain component get NAME --namespace default
  tanzu supplychain component get NAME --namespace default --show-details
```

## Options

```console
  -h, --help             help for get
  -n, --namespace name   kubernetes namespace
  -o, --output string    output the component details formatted. Supported formats: "json", "yaml", "yml"
      --show-details     show more details of the component
```

## Options inherited from parent commands

```console
      --kubeconfig file   kubeconfig file (default is $HOME/.kube/config)
      --no-color          deactivate color, bold, animations, and emoji output
```

## See also

- [tanzu supplychain component](tanzu_supplychain_component.hbs.md) for interacting with supplychain
  components