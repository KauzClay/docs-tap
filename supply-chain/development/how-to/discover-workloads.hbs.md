# Work with workloads

This topic tells you how to work with workloads when using Tanzu Supply Chain.

{{> 'partials/supply-chain/beta-banner' }}

This topic explains how to:

- Find the kinds of workloads you can use
- Create and delete a workload
- Observe runs of your workloads

## <a id="find-kinds-workloads"></a> Find the kinds of workloads you can use

Use the Tanzu Workload CLI plug-in to see available `Workload` kinds by running:

```console
tanzu workload kinds list
```

Example:

```console
KIND                        VERSION   DESCRIPTION
buildserverapps.vmware.com  v1        Builds a Kubernetes deployment app exposed using a Service using Buildpacks.
buildwebapps.vmware.com     v1        Pulls the source code from a Git repository and builds a Knative Service app using Buildpacks.
buildworkerapps.vmware.com  v1        Creates a background worker app deployed as Kubernetes deployment using Buildpacks.

🔎 To generate a workload for one of these kinds, use 'tanzu workload generate'
```

## <a id="create-and-delete"></a> Create and delete a workload

In this section you:

- Generate a workload manifest
- Create a workload
- Apply a workload
- Delete a workload

### <a id="gen-workload-manifest"></a> Generate a workload manifest

To generate a workload manifest:

1. Generate a `Workload` manifest with default configuration by running:

   ```console
   tanzu workload generate APP-NAME --kind buildwebapps.vmware.com
   ```

   Where `APP-NAME` is the name of the app. For example, `my-web-app`.

   Example:

   ```console
   apiVersion: vmware.com/v1
   kind: BuildWebApp
   metadata:
     name: my-web-app
   spec:
     source:
       #! Use this object to retrieve source from a git repository.
       #! The tag, commit, and, branch fields are mutually exclusive, use only one.
       #! Required
       git:
         #! A git branch ref to watch for new source
         branch: ""
         #! A git commit sha to use
         commit: ""
         #! A git tag ref to watch for new source
         tag: ""
         #! The url to the git source repository
         #! Required
         url: ""
       #! path inside the source to build from (build has no access to paths above the subPath)
       subPath: ""
   ...
   #! other configuration
   ```

1. Store the `Workload` manifest in a file for use by the `tanzu workload create`,
   `tanzu workload apply`, and `tanzu workload get` commands, and pipe the output into a `workload.yaml`
   file, by running:

   ```console
   tanzu workload generate APP-NAME --kind buildwebapps.vmware.com > workload.yaml
   ```

   Where `APP-NAME` is the name of the app. For example, `my-web-app`.

### <a id="create-workload"></a> Create a workload

To create a workload:

1. Create a `Workload` on the cluster from a manifest by running:

   ```console
   tanzu workload create --file workload.yaml --namespace build
   ```

   Example:

   ```console
   Creating workload:
         1 + |---
         2 + |apiVersion: vmware.com/v1
         3 + |kind: BuildWebApp
         4 + |metadata:
         5 + |  name: <my-web-app>
         6 + |  namespace: build
         7 + |spec:
         8 + |  source:
         9 + |    git:
       10 + |      branch: ""
       11 + |      commit: ""
       12 + |      tag: ""
       13 + |      url: ""
       14 + |    subPath: ""
       15 + |  ...
   Create workload my-web-app from workload.yaml? [yN]: y
   Successfully created workload my-web-app
   ```

1. Override the `Workload` name provided in the manifest by running:

   ```console
   tanzu workload create NAME --file workload.yaml --namespace build
   ```

   Where `NAME` is a name to serve as an argument for the manifest. For example, `my-web-app-2`.

### <a id="apply-workload"></a> Apply a workload

To apply a workload:

1. The `tanzu workload create` command is only used to create a `Workload` that does not already
   exist. To update an existing `Workload`, use `tanzu workload apply`. Apply a `Workload` manifest
   to the cluster by running:

   ```console
   tanzu workload apply --file workload.yaml --namespace build
   ```

   Example:

   ```console
   Creating workload:
         1 + |---
         2 + |apiVersion: vmware.com/v1
         3 + |kind: BuildWebApp
         4 + |metadata:
         5 + |  name: my-web-app
         6 + |  namespace: build
         7 + |spec:
         8 + |  source:
         9 + |    git:
       10 + |      branch: ""
       11 + |      commit: ""
       12 + |      tag: ""
       13 + |      url: ""
       14 + |    subPath: ""
       15 + |  ...
   Create workload my-web-app from workload.yaml? [yN]: y
   Successfully created workload my-web-app
   ```

1. Override the `Workload` name provided in the manifest by running:

   ```console
   tanzu workload apply NAME --file workload.yaml --namespace build
   ```

   Where `NAME` is a name to serve as an argument for the manifest. For example, `my-web-app-2`.

### <a id="delete-workload"></a> Delete a workload

Delete a `Workload` by name within a namespace by running:

```console
tanzu workload delete --file /tmp/workload.yaml --namespace build
```

Example:

```console
Really delete the workload my-web-app of kind buildwebapps.vmware.com from the build namespace? [yN]: y
Successfully deleted workload my-web-app
```

> **Note** Deleting a `Workload` prevents new builds while preserving built images in the registry.

## <a id="observe-runs"></a> Observe the runs of your workload

Use the Tanzu Workload CLI plug-in to observe `Workloads` and their `WorkloadRuns`:

1. List all workloads on the cluster by running:

   ```console
   tanzu workload list --namespace build
   ```

   Example:

   ```console
   Listing workloads from the build namespace

     NAME        KIND                     VERSION  AGE
     my-web-app  buildwebapps.vmware.com  v1       6m54s
   ```

1. Get the details of the specified `Workload` within a namespace by running:

   ```console
   tanzu workload get APP-NAME --namespace build
   ```

   Where `APP-NAME` is the application name. For example, `my-web-app`.

   Example:

   ```console
   $ tanzu workload get my-web-app --namespace build
   Overview
     name:       my-web-app
     kind:       buildwebapps.vmware.com/my-web-app
     namespace:  build
     age:        17s

   Runs:
     ID                    STATUS   DURATION  AGE
     my-web-app-run-lxwrm  Running  0s        17s
   ```

1. Get the workload output as YAML for programmatic use by running:

   ```console
   tanzu workload get APP-NAME --namespace build -o yaml
   ```

   Where `APP-NAME` is the application name. For example, `my-web-app`.

   Example:

   ```console
   $ tanzu workload get my-web-app --namespace build -o yaml
   ---
   apiVersion: vmware.com/v1
   kind: BuildWebApp
   metadata:
     name: my-web-app
     namespace: build
   spec:
     ...
   ```

1. Get the details of the specified workload run within a namespace by running:

   ```console
   tanzu workload run get APP-NAME-run-lxwrm -n build --show-details
   ```

   Where `APP-NAME` is the application name. For example, `my-web-app`.

   Example:

   ```console
   Overview
     name:        my-web-app
     kind:        buildwebapps.vmware.com/my-web-app
     run id:      buildwebappruns.vmware.com/my-web-app-run-lxwrm
     status:      Running
     namespace:   build
     age:         39s

   Spec
         1 + |---
         2 + |apiVersion: vmware.com/v1
         3 + |kind: BuildWebApp
         4 + |metadata:
         5 + |  name: my-web-app
         6 + |  namespace: build
         7 + |spec:
         8 + |...

   Stages
       ├─ source-git-provider
       │  ├─ check-source - Success
       │  │  ├─ Duration: 6s
       │  │  └─ Results
       │  │     ├─ message: using git-branch: main
       │  │     ├─ sha: <image SHA>
       │  │     └─ url: <image URL>
       │  └─ pipeline - Success
       │     ├─ Duration: 1m38s
       │     └─ Results
       │        ├─ url: <image URL>
       │        └─ digest: <image SHA>
       ├─ buildpack-build
       │  ├─ check-builders - Success
       │  │  ├─ Duration: 5s
       │  │  └─ Results
       │  │     ├─ builder-image: <image URL>
       │  │     ├─ message: Builders resolved
       │  │     └─ run-image: <image URL>
       │  └─ pipeline - Success
       │     ├─ Duration: 50s
       │     └─ Results
       │        ├─ url: <image URL>
       │        └─ digest: <image SHA>
       ├─ conventions
       │  └─ pipeline - Running
       │     └─ Duration: 53.693499s
       ├─ app-config-web
       │  └─ pipeline - Not Started
       ├─ carvel-package
       │  └─ pipeline - Not Started
       └─ git-writer-pr
          └─ pipeline - Not Started
   ```