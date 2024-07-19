# tanzu supplychain list

This topic gives you command-specific information for the Tanzu Supply Chain CLI.

## Synopsis

Lists all supplychains

list configuration options include:

- Output the list for all namespaces
- Output the list for a specific namespace
- Output the list formatted

```console
tanzu supplychain list [flags]
```

## Examples

```console
tanzu supplychain list
  tanzu supplychain list --all-namespaces
```

## Options

```console
  -A, --all-namespaces   use all kubernetes namespaces
  -g, --group string     list all supplychains from a specific group
  -h, --help             help for list
  -n, --namespace name   kubernetes namespace (defaulted from kube config)
  -o, --output string    output the supplychains formatted. Supported formats: "json", "yaml", "yml"
  -w, --wide             output the supplychain list with additional information
```

## Options inherited from parent commands

```console
      --kubeconfig file   kubeconfig file (default is $HOME/.kube/config)
      --no-color          deactivate color, bold, animations, and emoji output
```

## SEE ALSO

- [tanzu supplychain](tanzu_supplychain.hbs.md) - supplychain management