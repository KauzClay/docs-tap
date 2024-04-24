# Build your first Component

This topic tells you how to use Tanzu Supply Chain to build a Component.

{{> 'partials/supply-chain/beta-banner' }}

## Prerequisite

You must have a `SupplyChain` resource. For how to build one, see
[Build your first SupplyChain](my-first-supply-chain.hbs.md).

## Getting started

In [Build your first Supply Chain](my-first-supply-chain.hbs.md), you built a `SupplyChain` resource
that retrieves the source code from a Git repository, and builds and packages it as a Carvel package.
The `SupplyChain` resource then initiates a pull request to transfer the Carvel package to
a GitOps repository, which enables the installation of the built package on the Run clusters.

This topic tells you how to create a component that you can use in your `SupplyChain` resource for
unit-testing your Maven apps. You will create a component called `maven-unit-tester`, which will run
unit tests on the source pulled by the `source-git-provider` stage.

```console
tanzu supplychain component get source-git-provider-1.0.0 --show-details
```

Example:

```console
...
📤 Outputs
   git
    ├─ digest: $(resumptions.check-source.results.sha)
    ├─ type: git
    └─ url: $(resumptions.check-source.results.url)
   source
    ├─ digest: $(pipeline.results.digest)
    ├─ type: source
    └─ url: $(pipeline.results.url)
...
```

## Create a Tekton `Pipeline`

The first step for creating a `Component` resource is to make a Tekton `Pipeline` resource that
includes the logic the component uses. In this case, `maven-unit-tester`:

- Obtains the source code extracted by the `source-git-provider` component.
- Uses the source code as input.
- Runs the unit test command on the source repository.

Tekton `Pipeline`:

- Defines `parameters` that a developer can provide by using the workload.
- Creates a shared `workspace` to store intermediate files, such as the source code from the
  `source-git-provider` stage.
- Provides tasks that run the unit-testing logic.

To create a Tekton `Pipeline`:

1. Define `parameters` as shown in the following YAML:

    ```yaml
    ---
    apiVersion: tekton.dev/v1
    kind: Pipeline
    metadata:
        name: maven-unit-tester-1.0.0
    spec:
        description: Test maven application using mvn test command
        params:
          - description: url of source registry image
            name: source-url
            type: string
          - description: sub directory in source to become root dir for build
            name: sub-path
            default: "."
            type: string
          - description: The test command to run in my task
            name: testcmd
            type: string
          - description: The test command to run in my task
            name: workload-name
            type: string
    ```

1. Define a Tekton workspace as shown in the following YAML:

    ```yaml
        workspaces:
          - name: shared-data
    ```

1. Add two tasks to the `Pipeline` resource. Task 1 pulls the output from `source-git-provider` and
   puts it in the workspace `shared-data` that was defined in the `Pipeline` resource earlier. Task 2
   runs the unit tests and records the output as a Tekton Result.

   See the following example YAML:

    ```yaml
        tasks:
          # task 1 pulls the output from source-git-provider and puts it in a workspace called shared-data
          - name: fetch-source
            params:
              - name: url
                value: $(params.source-url)
            taskRef:
              kind: Task
              name: fetch-tgz-content-oci
            workspaces:
              - name: store
                workspace: shared-data

          # task 2 runs the unit tests
          - name: test
            runAfter:
              - fetch-source
            params:
              - name: sub-path
                value: $(params.sub-path)
              - name: testcmd
                value: $(params.testcmd)
              - name: workload-name
                value: $(params.workload-name)
            taskSpec:
              params:
                - name: sub-path
                - name: testcmd
                - name: workload-name
              results:
                - description: Error message if things failed
                  name: message
                  type: string
              steps:
                - name: test
                  image: maven
                  env:
                    - name: HOME
                      value: /tekton/home/
                  script: |-
                    #!/usr/bin/env sh
                    set -eu
                    ls -al $(workspaces.store.path)
                    cd $(workspaces.store.path)/$(params.sub-path)

                    set +e
                    $(params.testcmd)
                    testval=$?
                    set -e

                    if [ $testval -ne 0 ]; then
                      echo "Unit tests stage failed. Please run 'tanzu workload logs $(params.workload-name)' to see the run logs for more details." | tee "$(results.message.path)"
                      exit $testval
                    fi
              stepTemplate:
                  computeResources: {}
                  securityContext:
                      allowPrivilegeEscalation: false
                      capabilities:
                          drop:
                              - ALL
                      runAsNonRoot: true
                      runAsUser: 1001
                      seccompProfile:
                          type: RuntimeDefault
            workspaces:
              - name: store
                workspace: shared-data

    ```

1. Add the version of the pipeline in the name of the YAML file as well as the name of the pipeline.
   This enables you to create new versions of the pipelines without interfering with the old one.

1. Store the `Pipeline` resource in the `pipelines` directory created by the CLI as
   `maven-unit-tester-1.0.0.yaml`, as seen in the following YAML:

    ```yaml
    ---
    apiVersion: tekton.dev/v1
    kind: Pipeline
    metadata:
        name: maven-unit-tester-1.0.0
    spec:
        description: Test maven application using mvn test command
        params:
          - description: url of source registry image
            name: source-url
            type: string
          - description: sub directory in source to become root dir for build
            name: sub-path
            default: "."
            type: string
          - description: The test command to run in my task
            name: testcmd
            type: string
          - description: The test command to run in my task
            name: workload-name
            type: string
        workspaces:
          - name: shared-data
        # My tasks starts here
        tasks:
          # task 1 pulls the output from source-git-provider and puts it in a workspace called shared-data
          - name: fetch-source
            params:
              - name: url
                value: $(params.source-url)
            taskRef:
              kind: Task
              name: fetch-tgz-content-oci
            workspaces:
              - name: store
                workspace: shared-data

          # task 2 runs the unit tests
          - name: test
            runAfter:
              - fetch-source
            params:
              - name: sub-path
                value: $(params.sub-path)
              - name: testcmd
                value: $(params.testcmd)
              - name: workload-name
                value: $(params.workload-name)
            taskSpec:
              params:
                - name: sub-path
                - name: testcmd
                - name: workload-name
              results:
                - description: Error message if things failed
                  name: message
                  type: string
              steps:
                - name: test
                  image: maven
                  env:
                    - name: HOME
                      value: /tekton/home/
                  script: |-
                    #!/usr/bin/env sh
                    set -eu
                    ls -al $(workspaces.store.path)
                    cd $(workspaces.store.path)/$(params.sub-path)

                    set +e
                    $(params.testcmd)
                    testval=$?
                    set -e

                    if [ $testval -ne 0 ]; then
                      echo "Unit tests stage failed. Please run 'tanzu workload logs $(params.workload-name)' to see the run logs for more details." | tee "$(results.message.path)"
                      exit $testval
                    fi
              stepTemplate:
                  computeResources: {}
                  securityContext:
                      allowPrivilegeEscalation: false
                      capabilities:
                          drop:
                              - ALL
                      runAsNonRoot: true
                      runAsUser: 1001
                      seccompProfile:
                          type: RuntimeDefault
            workspaces:
              - name: store
                workspace: shared-data

    ```

## Create a `SupplyChain` `Component`

The section describes how to write a `Component` manifest that defines the following:

- Configuration for a `Component` that is exposed as part of the Developer API. This is the
  configuration that a platform engineer can override at a `SupplyChain` level or that a developer
  can provide by using the `Workload` if a platform engineer does not override it at the
  `SupplyChain` level.

- `Inputs`, which is a list of named `Outputs` from a previous stage in a `SupplyChain` resource that
  the current `Component` resource depends on.

- `Outputs`, which are named outputs from this component that can be used by another `Component`
  resource as `Inputs`.

- The Pipeline Run Definition section, which defines the work done by this component.
  `spec.pipelineRun` is used to create a Tekton `PipelineRun` and has many similarities.

1. Define the `Component` resource configuration as shown in the following YAML:

    ```yaml
    ---
    apiVersion: supply-chain.apps.tanzu.vmware.com/v1alpha1
    kind: Component
    metadata:
        name: maven-unit-tester-1.0.0
    spec:
        description: Execute a maven unit test command using the mvn CLI
        config:
            - path: spec.test.cmd
              schema:
                description: |
                    Unit testing mvn command to run
                required:
                    - cmd
                type: string
                default: "mvn test"
                example: "mvn test"
    ```

1. Define `Component` inputs and outputs. In this case, the output of the `source-git-provider`
   component is used as an input for the `maven-unit-tester` component.

    ```yaml
        inputs:
          - name: source
            type: source
    ```

1. Define `Component` `PipelineRun` details. The `Pipeline` resource created earlier is referenced
   here. The values are passed from the `Workload` and `Inputs` to satisfy the parameters for the
   `Pipeline` resource. See the following example YAML:

    ```yaml
        pipelineRun:
            params:
                - name: source-url
                  value: $(inputs.source.url)
                - name: sub-path
                  value: $(workload.spec.source.subPath)
                - name: testcmd
                  value: $(workload.spec.test.cmd)
                - name: workload-name
                  value: $(workload.metadata.name)
            pipelineRef:
                name: maven-unit-tester-1.0.0
            taskRunTemplate:
                podTemplate:
                    securityContext:
                        fsGroup: 1000
                        runAsGroup: 1000
                        runAsUser: 1001
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

1. Store the `Component` resource in the `components` directory created by the CLI as
   `maven-unit-tester-1.0.0.yaml`. See the following example YAML:

    ```yaml
    ---
    apiVersion: supply-chain.apps.tanzu.vmware.com/v1alpha1
    kind: Component
    metadata:
        name: maven-unit-tester-1.0.0
    spec:
        description: Execute a maven unit test command using the mvn CLI
        inputs:
          - name: source
            type: source
        config:
            - path: spec.test.cmd
              schema:
                description: |
                    Unit testing mvn command to run
                required:
                    - cmd
                type: string
                default: "mvn test"
                example: "mvn test"
        pipelineRun:
            params:
                - name: source-url
                  value: $(inputs.source.url)
                - name: sub-path
                  value: $(workload.spec.source.subPath)
                - name: testcmd
                  value: $(workload.spec.test.cmd)
                - name: workload-name
                  value: $(workload.metadata.name)
            pipelineRef:
                name: maven-unit-tester-1.0.0
            taskRunTemplate:
                podTemplate:
                    securityContext:
                        fsGroup: 1000
                        runAsGroup: 1000
                        runAsUser: 1001
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

## Install the component

You can now use the `make install` target created by the `tanzu supplychain` CLI to install the new
`Component` and `Pipeline` resources to see if they reconcile successfully.

Install the component by running:

```console
NAMESPACE=mysupplychains make install
```

This command also brings up any errors that Tekton throws.

## Next Steps

Add your new `Component` to a `SupplyChain`. For instructions, see
[Add stages to your Supply Chain](add-stages-supply-chain.hbs.md).

## Useful links

- [Component API Reference](../../reference/api/component.hbs.md)
- [Component Catalog](../../reference/catalog/about.hbs.md)