# Component API

This topic gives you reference information about the `Component` resource for Tanzu Supply Chain.

{{> 'partials/supply-chain/beta-banner' }}

Components define the work to be done in one [Stage](supplychain.hbs.md#specstages) of the `SupplyChain`.

## Type and Object Metadata

```yaml
apiVersion: supply-chain.apps.tanzu.vmware.com/v1alpha1
kind: Component
```

### `metadata.name`

`metadata.name:` must have a `-M.m.p` suffix, representing the major, minor and patch of this
version of the component. Changes to the config section should coincide with a bump to major or
minor versions. Reserve patch increments for changes that do not alter the API, or the behavior
significantly.

```yaml
metadata:
  name: source-1.0.0
```

## Spec

### <a id='spec-config'></a>`spec.config`

`spec.config` defines the configuration in a workload (`spec` of the workload) that is required for
the component to operate.

`spec.config` is an array of objects. Each object has three fields:

- `path` describes the path in the workload where this configuration is appended or merged. It must
  start with `spec.`
- `schema` defines a property. It must be a valid OpenAPI v3 schema. For more information, see the
  [Kubernetes documentation](https://kubernetes.io/docs/home/) and the
  [OpenAPI documentation](https://swagger.io/specification/).
- `required` determines whether the property that `path` references is marked as required.

> **Note** Use the `required` field in the `spec.config` array to mark properties as required. You
> can mark child properties separately in the `schema`.

#### Example

Given the following `Component` `spec.config`:

```yaml
config:
  - path: spec.source.git
    schema:
      type: object
      description: |
        Fill this object in if you want your source to come from Git.
        The tag, commit, and branch fields are mutually exclusive.
        Use only one.
      properties:
        tag:
          description: A Git tag ref to watch for a new source
          type: string
        commit:
          description: A Git commit SHA to use
          type: string
        branch:
          description: A Git branch reference to watch for a new source
          type: string
        url:
          description: The URL to the Git source repository
          type: string
      required:
        - url
  - path: spec.image
    required: true
    schema:
      type: string
      description: Repository where the image is published
```

A `SupplyChain` that includes the `Component` defines the workload `spec` as follows in the
`CustomResourceDefinition`:

```yaml
properties:
  spec:
    type: object
    properties:
      image:
        type: string
        description: Repository where the image is published
      source:
        type: object
        description: |
          Fill this object in if you want your source to come from Git.
          The tag, commit, and branch fields are mutually exclusive.
          Use only one.
        properties:
          branch:
            type: string
            description: A Git branch reference to watch for a new source
          commit:
            type: string
            description: A Git commit SHA to use
          tag:
            type: string
            description: A Git tag reference to watch for a new source
          url:
            type: string
            description: The URL to the Git source repository
        required:
          - url
    - path: spec.source.subPath
      required: true    # makes subPath a required field
      schema:
        type: string
        description: the subPath of the repository to monitor for changes
```

### `spec.description`

`spec.description` describes the component's purpose.
You see this description in `tanzu workload run list`.

```yaml
spec:
  description: Gets the latest source and stores it in an OCI Image
```

### `spec.inputs`

The inputs this component requires.
This component cannot be added to a supply chain unless a previous stage exposes the name and type as
output.

```yaml
  inputs:
    - name: source
      type: source
```

### `spec.outputs`

The outputs this component emits. Outputs are references to an artifact located at the `url`.

URL string `json:"url,omitempty"`

```yaml
outputs:
  - name: source
    type: source
    digest: <cryptographic hash>
    url: <location of output artifact>
```

#### `spec.outputs[].digest`

The digest of this output. If not provided, the default is `$(pipeline.results.digest)`.

Use a template replacement to describe where the digest originates, in either the
pipeline or resumption. For example, `$(resumption.results.commitId)` or `$(pipeline.results.shasum)`.

This digest must be a [cryptographic hash function](https://en.wikipedia.org/wiki/Cryptographic_hash_function)
to assure operators of the supply chain that the value correlates to the data in the artifact at the
`url`. Describe how the digest is formed so that it can be verified at a later date if the
authenticity of the artifact needs to be checked.

#### `spec.outputs[].url`

The URL that holds the artifact of this output. If not provided, the default is `$(pipeline.results.url)`.

Use a template replacement to describe where the URL originates, typically in the pipeline. For
example, `$pipeline.results.buildURL`.

The storage of the resource at this URL must match in all instances of this `output.type`, including
any expected `accepts-*` headers and so on.

#### Example

```yaml
outputs:
  - name: source
    type: source
    digest: $(resumption.results.commitId)
    url: $(pipeline.results.buildURL)
```

### `spec.pipelineRun`

The `spec.pipelineRun` section defines the work that this component performs. Tanzu Supply Chain
uses `spec.pipelineRun` to create a Tekton `PipelineRun`, and there are many similarities between
Tekton `PipelineRun` and `spec.pipelineRun`.

#### `spec.pipelineRun.params`

`spec.pipelineRun.params` are the same as Tekton `PipelineRun` parameters except for one major
difference: You can populate `spec.pipelineRun.params` by using templates.

The available references for templating are:

| reference                                     | source                                           | examples                                                   |
|-----------------------------------------------|--------------------------------------------------|------------------------------------------------------------|
| `$(config.spec...)`                           | References to the [config](#spec-config)         | `$(config.spec.source.git.url)`                            |
| `$(workload.spec...)`                         | The same as `$(config.spec)...` - **Deprecated** | `$(workload.spec.source.git.url)`                          |
| `$(workload.metadata...)`                     | The workload metadata                            | `$(workload.metadata.labels)`, `$(workload.metadata.name)` |
| `$(inputs.<input-name>.[url\|digest])`        | An input URL or digest                           | `$(inputs.image.url)`, `$(inputs.image.digest)`            |
| `$(resumptions.<resumption-name>.results...)` | A [resumption](#specresumptions) result          | `$(resumptions.check-source.results.sha)`                  |

#### `spec.pipelineRun.pipelineRef`

The `spec.pipelineRun.pipelineRef` is required, and it has one field `name` that must refer to the
`metadata.name` of a Tekton Pipeline that resides in the same namespace as the `Component` and
`SupplyChain`.

### `spec.pipelineRun.taskRunSpecs`

If you need to define `taskRunSpecs` to pass to Tekton `PipelineRun`, use
`spec.pipelineRun.taskRunSpecs`. This is identical to the Tekton `PipelineRun` `taskRunSpecs`
specification.

### `spec.pipelineRun.taskRunTemplates`

If you need to define `taskRunTemplates` to pass to Tekton `PipelineRun`, use
`spec.pipelineRun.taskRunTemplates`. This is identical to the Tekton `PipelineRun` `taskRunTemplates`
specification.

#### `spec.pipelineRun.workspaces`

If you need to define workspaces to pass to Tekton `PipelineRun`, use `spec.pipelineRun.workspaces`.
This field is an array of workspace definitions, and is identical to the Tekton Workspaces specification.

| Reference                                     | Source                                                                                         | Examples                                                   |
|-----------------------------------------------|------------------------------------------------------------------------------------------------|------------------------------------------------------------|
| `$(workload.spec...)` (deprecated)            | The workload spec                                                                              | `$(workload.spec.source.git.url)`                          |
| `$(workload.metadata...)`                     | The workload metadata                                                                          | `$(workload.metadata.labels)`, `$(workload.metadata.name)` |
| `$(config.spec...)`                           | `config.spec` is derived from `workload.spec`, unless modified by `supplychain.spec.overrides` | `$(config.spec.source.git.url)`                            |
| `$(inputs.<input-name>.[url\|digest])`        | An input URL or digest                                                                         | `$(inputs.image.url)`, `$(inputs.image.digest)`            |
| `$(resumptions.<resumption-name>.results...)` | A [resumption](#specresumptions) result                                                        | `$(resumptions.check-source.results.sha)`                  |

#### Example

```yaml
spec:
  pipelineRun:
    pipelineRef:
      name: source
    params:
      - name: git-url
        value: $(config.spec.source.git.url)
      - name: sha
        value: $(resumptions.check-source.results.sha)
    workspaces:
      - name: shared-data
        volumeClaimTemplate:
          spec:
            accessModes:
              - ReadWriteOnce
            resources:
              requests:
                storage: 1Gi
```

### `spec.resumptions[]`

`spec.resumptions[]` define Tekton `TaskRuns`. They are optional, but useful to describe small, fast
tasks that check for dependency changes, such as new source, or new base images.

For a detailed explanation of resumptions, see
[Core Concepts: Resumptions](../../platform-engineering/explanation/resumptions.hbs.md).

#### `spec.resumptions[].name`

Make `spec.resumptions[].name` descriptive for the end users who will see it. Write it as
`check-RESOURCE-TYPE`, where `RESOURCE-TYPE` is the kind of resource being polled, such as `source`
or `base-image`.

#### `spec.resumptions[].trigger.runaAfter`

`spec.resumptions[].trigger.runaAfter` describes the rerun period for the task. The task is
executed, and after it completes (successfully or otherwise), Tanzu Supply Chain waits the
`runaAfter` period and then executes the task again. This continues indefinitely.

`runAfter` can be specified using the
[`time.ParseDuration()`](https://pkg.go.dev/maze.io/x/duration#ParseDuration) specification.

#### `spec.resumptions[].taskRef`

The `spec.resumptions[].taskRef` has one field `name` that must refer to the `metadata.name`
of a Tekton `Task` that resides in the same namespace as the `Component` and `SupplyChain`.

This is the task that run's on the resumptions `spec.resumptions[].trigger`

#### `spec.resumptions[].params`

`spec.resumptions[].params` are the same as Tekton `TaskRun` Parameters with one major difference:
you can populate them using templates.

The available references for templating references are:

| Reference                              | Source                                                                                         | Examples                                                   |
|----------------------------------------|------------------------------------------------------------------------------------------------|------------------------------------------------------------|
| `$(workload.spec...)` (deprecated)     | The workload spec                                                                              | `$(workload.spec.source.git.url)`                          |
| `$(workload.metadata...)`              | The workload metadata                                                                          | `$(workload.metadata.labels)`, `$(workload.metadata.name)` |
| `$(config.spec...)`                    | `config.spec` is derived from `workload.spec`, unless modified by `supplychain.spec.overrides` | `$(config.spec.source.git.url)`                            |
| `$(inputs.<input-name>.[url\|digest])` | An input URL or digest                                                                         | `$(inputs.image.url)`, `$(inputs.image.digest)`            |

#### Example

```yaml
  resumptions:
    - name: check-source
      trigger:
        runAfter: 60s
      taskRef:
        name: check-source
      params:
        - name: git-branch
          value: $(config.spec.source.git.branch)
        - name: git-url
          value: $(config.spec.source.git.url)
        - name: git-commit
          value: $(config.spec.source.git.commit)
        - name: git-tag
          value: $(config.spec.source.git.tag)
```

## Status

### `status.conditions[]`

Every `status.conditions[]` in Tanzu Supply Chain resources follows a
[strict set of conventions](statuses.hbs.md)

`Component` resources are "living", however they are resistant to changes in their spec, They're
designed to be immutable on production servers, so that accidental spec changes do not break the API
delivered to end users.

If a `Component`'s top level condition `Ready` is ever something other than `status: "True"` then
the `reason` field should describe the problem with the component.

### `status.details`

`status.details` describe some of the observations that Tanzu Supply Chains made about this
component. These fields are used to improve the output of `kubernetes get component <name> -owide`,
and they summarize the component for platform engineers when they author a `SupplyChain`.

```yaml
status:
  conditions:
  - lastTransitionTime: "2024-02-25T00:40:46Z"
    message: ""
    reason: Ready
    status: "True"
    type: Ready
  details:
    hasResumptions: "False"
    inputs: conventions[conventions[]
    outputs: oci-yaml-files[oci-yaml-files[], oci-ytt-files[oci-ytt-files[]
  observedGeneration: 1
```

### <a id='status-docs'></a>`status.docs`

`status.docs` contains a human-readable explanation of the content of the component.
It's useful for CLIs and UIs, and provides a clean summary of the component.

#### Example

```
docs: |
  # source

  Version: 1.0.0

  ## Description

  Monitors a git repository

  ## Inputs

  * _none_

  ## Outputs

  | Name  | Type |
  | ---   | ---  |
  | source | [source](./output-types.hbs.md#source) |

  ## Config

  ``yaml
  spec:
    source:
      # Fill this object in if you want your source to come from git.
      # The tag, commit and branch fields are mutually exclusive, use only one.
      git:
        # A git branch ref to watch for new source
        branch:
        # A git commit sha to use
        commit:
        # A git tag ref to watch for new source
        tag:
        # The url to the git source repository
        # +required
        url:
  ``
```

<!--
[SupplyChain]: supplychain.hbs.md
[Workload]: workload.hbs.md
[Component]: ./component.hbs.md
[Components]: ./component.hbs.md
[WorkloadRun]: workloadrun.hbs.md
[OpenAPI Structural Schema]: https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#specifying-a-structural-schema
[Tekton PipelineRun]: https://tekton.dev/docs/pipelines/pipelineruns/
[Tekton PipelineRuns]: https://tekton.dev/docs/pipelines/pipelineruns/
[Tekton Pipeline]: https://tekton.dev/docs/pipelines/pipelines/
[Tekton Pipelines]: https://tekton.dev/docs/pipelines/pipelines/
[Tekton Workspaces]: https://tekton.dev/docs/pipelines/pipelineruns/#specifying-workspaces
[Tekton PipelineRun Parameters]: https://tekton.dev/docs/pipelines/pipelineruns/#specifying-parameters
[Tekton TaskRun Parameters]: https://tekton.dev/docs/pipelines/taskruns/#specifying-parameters
[Tekton Taskruns]: https://tekton.dev/docs/pipelines/taskruns/
[Tekton Task]: https://tekton.dev/docs/pipelines/tasks/
-->