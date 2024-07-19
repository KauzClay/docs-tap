# tanzu supplychain get

This topic gives you command-specific information for the Tanzu Supply Chain CLI.

## Synopsis

Get the details of the specified supplychain within a namespace.

`supplychain get` configuration options include:

- Specify the namespace of the supplychain
- Output the supplychain formatted

```console
tanzu supplychain get <NAME> [flags]
```

## Examples

```console
tanzu supplychain get NAME
```

## Options

```console
  -h, --help             help for get
  -n, --namespace name   kubernetes namespace (defaulted from kube config)
  -o, --output string    output the supplychains formatted. Supported formats: "json", "yaml", "yml"
```

## Options inherited from parent commands

```console
      --kubeconfig file   kubeconfig file (default is $HOME/.kube/config)
      --no-color          deactivate color, bold, animations, and emoji output
```

## SEE ALSO

- [tanzu supplychain](tanzu_supplychain.hbs.md) - supplychain management