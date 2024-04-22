# Build your first Component

This topic tells you how to build a Component using Tanzu Supply Chain.

{{> 'partials/supply-chain/beta-banner' }}

## Prerequisite

- Ensure that you created a SupplyChain by following the tutorial [Build your first SupplyChain](./my-first-supply-chain.hbs.md).

## Getting started

In the first tutorial, we built a SupplyChain that retrieves the source code from a Git repository, and proceeds with building and
packaging it as a Carvel package. Subsequently, it initiates a pull request to transfer the Carvel
package to a GitOps repository, facilitating the installation of the built package on the Run clusters. 

In this tutorial, We will create a component that we can use in our supplychain for unit testing our Maven apps. 
We will create a component called `maven-unit-tester` and that will run unit tests on the source pulled by the `source-git-provider` stage.

```
tanzu supplychain component get source-git-provider-1.0.0 --show-details
```
Example: 
```
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

## Create a Tekton pipeline

### Step 1: Define Parameters
```
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

### Step 2: Define a Tekton workspace

```
    workspaces:
      - name: shared-data
```

### Step 3: Add Tasks to the pipeline
We will add 2 tasks in the `Pipeline` resource.

* **Task 1:** will pull the output from source-git-provider and puts it in a workspace `shared-data` we defined in our pipeline above.
* **Task 2:** will run the unit tests and record the output as a Tekton Result.

```
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

Put it all togather and our pipeline looks as follows:

```
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

## Create a Supplychain Component

### Step 1: Define Component Configuration
```
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

### Step 2: Define Component Inputs/Outputs

```
    inputs:
      - name: source
        type: source
```

### Step 3: Define Component Pipeline Details

```
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

Putting it all togather and your `Component` CR looks like follows:

```
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

```
NAMESPACE=mysupplychains make install
```

## Next Steps

Add your new `Component` to a `SupplyChain`. For instructions, see
[Add Stages to your Supply Chain](./add-stages-supply-chain.hbs.md).

## Useful links

- [Component API Reference](../../reference/api/component.hbs.md)
- [Component Catalog](../../reference/catalog/about.hbs.md)