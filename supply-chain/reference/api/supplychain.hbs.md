# SupplyChain API

This topic gives you reference information about the `SupplyChain` resource for Tanzu Supply Chain.

{{> 'partials/supply-chain/beta-banner' }}

The supply chain defines the object kind of the `Workload`, the `Components` used, and their order.

## Type and Object Metadata

```yaml
apiVersion: supply-chain.apps.tanzu.vmware.com/v1alpha1
kind: SupplyChain
```

### `metadata.name`

`metadata.name` is in the form:

`<plural-name>.<group>-<Major>.<minor>.<patch>`

- `plural-name` must match the plural form of `defines.kind`, without the version.
  For example: `kind: JavaServerAppV3` would have a `plural-name` of `javaserverappv3s`
- `group` must match `defines.group`. See [spec.defines.group](#specdefines) later in the topic.
- `<Major>.<minor>.<patch>` is the version definition.

```yaml
metadata:
  name: hostedapps.widget.com-0.0.1
```

## Spec

### <a id='spec-config'></a>`spec.config`

The [spec.stages](#specstages) structure introduces `Components`, where the aggregated
[config sections](component.hbs.md#specconfig) form the `SupplyChain`'s configuration and become the
[Workload spec](workload.hbs.md#spec).

This configuration can be altered by changing defaults, or overridden and hidden from the `Workload`.

#### <a id='spec-config-defaults'></a>`spec.config.defaults`

Change or add defaults for the config that is presented in the
[Workload's spec](workload.hbs.md#spec).

`spec.config.defaults` is an array of `path:` and `value:` fields, where:

- `path` is the path to the configuration value, formatted as either:
  - The full path to the field you want to set.
  - The path to any structure where all desired child fields must be set.
- `value`: A string or structure value.

#### <a id='spec-config-overrides'></a>`spec.config.overrides`

Override the value of a config field, so that it no longer appears in the
[Workload spec](workload.hbs.md#spec).

`spec.config.overrides` is an array of `path:` and `value:` fields, where:

- `path` is the path to the configuration value, formatted as either:

  - The full path to the field you wish to set.
  - The path to any structure where all desired child fields must be set.

- `value` is the string or structure value.

#### Example

```yaml
apiVersion: supply-chain.apps.tanzu.vmware.com/v1alpha1
kind: SupplyChain
spec:
  config:
    defaults:
      - path: spec.source
        value:
          branch: develop
          url: https://our.git.com/our-monorepo
    overrides:
      - path: spec.registry.repository
        value: "YOUR-REGISTRY-REPO"
      - path: spec.registry.server
        value: "YOUR-REGISTRY-SERVER"
```

### `spec.defines`

The `spec.defines` object defines the `Workload` custom resource definition (CRD).

`spec.defines.group` (required) is used to fill in the `group` field in the
[CustomResourceDefinitionSpec].

`spec.defines.group` is the classic domain-formatted group of any Kubernetes object.
Use your organization's top-level domain or a departmental domain.

`spec.defines.kind` (required) is the name of the resource in CamelCase.

`spec.defines.plural` (required) is typically the plural form of the kind.
It must be all lowercase.

VMware recommends that you pluralize the name after the version. For example, change `WebAppV1` to
`webappv1s`.

`spec.defines.singular` is optional and is the lowercase of `kind` by default. For example,
`ServerAppv1` becomes `serverappv1`.

`spec.defines.shortnames` is a list and is empty by default. Use this to specify an array of aliases
for your kind. These are great for simplifying `kubectl` commands.

#### `spec.defines.categories`

`spec.defines.categories` is a list and is empty by default. `spec.defines.categories` specify a
collection term for a group of kinds, so that `kubectl get <category>` returns instances of all
kinds in the category.

### Example

```console
spec:
  defines: # Describes the workload
    kind: HostedApp
    pluralName: hostedapps
    group: example.com
    version: v1alpha1
    categories:
      - apps    # The kubectl get apps command includes this kind
                # and others with the same category.
    shortnames:
      - hosted1
      - ha1     # The kubectl get ha1 -A command shows all instance of this kind
```

### `spec.description`

An app developer can see the `spec.description` field when they use the CLI to discover available
`Workload` kinds. For example:

```console
$ tanzu workload kind list --wide

KIND                      VERSION     AGE   DESCRIPTION
serverappv2.example.com   v1alpha1    12m   Server application supply chain
```

Embed complete documentation in the description to best assist app developers.

The description field supports multiline plain text or Markdown.

### `spec.stages[]`

`spec.stages` break the work to be done by this supply chain into a serial collection of "stages",
each with a component.

This is where you define the operations of this `SupplyChain`.

### `spec.stages[].name`

Each stage has a `name`, which is shown to the user in the CLI and UI.
`name` can only be composed of hyphens and lowercase alphanumeric characters.

### `spec.stages[].componentRef`

Each stage also has a `componentRef` with a single field, `name`.
`componentRef.Name` refers to the name of a Tanzu Supply Chain [Component] resource.
Currently, [Component] must exist in the same namespace as the `SupplyChain`. For more information,
see the Tanzu Supply Chain [known issues](../../../release-notes.hbs.md#1-10-0-supply-chain-ki).

The supply chain returns an error if a component expects an input that has not been output by a
previous stage.

### <a id='security-context'></a> `spec.stages[].securityContext`

Each stage can specify a `securityContext` with a single field, `runAs`. When this is set to
`workload` the stage is executed in the namespace of the Workload rather than the Supply Chain. For
more information, see
[Security Model](../../platform-engineering/explanation/security-model.hbs.md).

#### Example

```yaml
apiVersion: supply-chain.apps.tanzu.vmware.com/v1alpha1
kind: SupplyChain
spec:
  stages:
    - name: fetch-source
      componentRef:
        name: source-git-provider-1.0.0
      securityContext:
        runAs: workload
    - name: build
      componentRef:
        name: golang-builder-1.0.0
    - name: commit
      componentRef:
        name: commit-writer-1.0.0
```

## Status

### `status.conditions[]`

Every `status.conditions[]` in Tanzu Supply Chain resources follows a
[strict set of conventions](statuses.hbs.md).

The top-level condition type is `Ready` because `SupplyChain` is a living resource.

The sub-types are:

#### RBACDefined

| Reason          | Meaning                                                                                                                                                    |
|-----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `Ready`         | The `RoleBinding`s for the kind declared in [spec.defines](#specdefines) was created on the cluster.                                                       |
| `AlreadyExists` | The `RoleBinding` record already exists.<br/>Most common cause of this issue is another Supply Chain with the same [`spec.defines`](#specdefines) section. |
| `UnknownError`  | The `RoleBinding` record failed due to an exceptional error. Look at the reconciler logs and contact Tanzu Support.                                        |

#### APIsDefined

| Reason          | Meaning                                                                                                                                                                         |
|-----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `Ready`         | The CRD for the kind declared in [spec.defines](#specdefines) was created on the cluster.                                                                                       |
| `Conflict`      | The CRD already exists and is managed by another SupplyChain.<br/>Most common cause of this issue is another Supply Chain with the same [`spec.defines`](#specdefines) section. |
| `Invalid`       | The CRD is invalid<br/>Most common cause of this is an illegal OpenAPIV3Schema in the [Component].                                                                              |
| `Unknown error` | The CRD could not be created due to an exceptional error. Look at the reconciler logs and contact Tanzu Support.                                                                |

#### StageMapping

| Reason                        | Meaning                                                                                                                                      |
|-------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| `Ready`                       | The [stages](#specstages) pass all the [validation rules](../../platform-engineering/explanation/supply-chains.hbs.md#integrity-validation). |
| `NoSuchComponent`             | The referenced component cannot be found.                                                                                                    |
| `NoSuchInput`                 | The input to the stage is not emitted by any stage.                                                                                          |
| `InputNotSatisfiedUntilLater` | The input to the stage is not emitted by a previous stage.                                                                                   |
| `InputMismatch`               | The input matches a previous output by name, however the type does not match.                                                                |
| `OutputRedefined`             | The output redefines an existing output. Shadowing of outputs is not supported.                                                              |

<!--
[Workload]: workload.hbs.md
[WorkloadRun]: workloadrun.hbs.md
[Components]: component.hbs.md
[Component]: component.hbs.md
[Input]: component.hbs.md#inputs
[Output]: component.hbs.md#outputs
[Object Kind]: https://kubernetes.io/docs/concepts/overview/working-with-objects/ "Kubernetes documentation for Objects"
[CRD]: https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/ "Kubernetes Custom Resource documentation"
[CustomResourceDefinition]: https://kubernetes.io/docs/reference/kubernetes-api/extend-resources/custom-resource-definition-v1/ "Kuberneted Custom Resource Definition API specification"
[CustomResourceDefinitionSpec]: https://kubernetes.io/docs/reference/kubernetes-api/extend-resources/custom-resource-definition-v1/#CustomResourceDefinitionSpec "Kuberneted CRD Spec API specification"
-->