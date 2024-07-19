# Tutorial: Deploy your first workload

This topic tells you how to use the Tanzu Workload CLI plug-in to create your first `Workload`.

{{> 'partials/supply-chain/beta-banner' }}

In this tutorial, the platform engineer has already created some Supply Chains for you to use.
The Supply Chains can pull the source code from the source repository and build it.
The built artifact is then pushed to a GitOps repository that the platform engineer selected.

## <a id="prepare"></a> Prepare

Install:

- [Tanzu CLI](../../../install-tanzu-cli.hbs.md#install-tanzu-cli)
- [Tanzu Workload CLI plug-in](../how-to/install-the-cli.hbs.md)

## <a id="deploy-workload"></a> Deploy a workload

To get started:

1. See which `SupplyChain` resources are available to you, and which kinds of `Workload` resources
   you can create by using those `SupplyChain` resources, by running:

   ```console
   tanzu workload kind list
   ```

   Example output:

   ```console
   $ tanzu workload kind list

     KIND                                       VERSION   DESCRIPTION
     appbuildv1s.supplychains.tanzu.vmware.com  v1alpha1  Supply chain that pulls the source code from git repo, builds it using
                                                          buildpacks and package the output as Carvel package.

   🔎 To generate a workload for one of these kinds, use 'tanzu workload generate'
   ```

   The command output shows that you have a `appbuildv1s.supplychains.tanzu.vmware.com` kind that
   you can use to generate the workload. The Tanzu CLI also shows a hint about the next command to
   use in the process.

1. For this tutorial, build your workload by using the
   [tanzu-java-web-app](https://github.com/vmware-tanzu/application-accelerator-samples/tree/main/tanzu-java-web-app)
   sample application.

   Use the `tanzu workload generate` command to create a `Workload` of the kind
   `AppBuildV1`. A drop-down menu is displayed if multiple kinds are available. If a single kind is
   available, it is used to generate the scaffold of the `Workload`. Run:

   ```console
   tanzu workload generate tanzu-java-web-app
   ```

   Example output:

   ```console
   apiVersion: supplychains.tanzu.vmware.com/v1alpha1
   kind: AppBuildV1
   metadata:
     name: tanzu-java-web-app
   spec:
     #! Configuration for the generated Carvel Package
     carvel:
       ...
     gitOps:
       ...
     #! Configuration for the registry to use
     registry:
       ...
     source:
       ...
     #! Kpack build specification
     build:
       ...
   ```

1. Pipe the output into a `workload.yaml` file. If you have more than one kind available in the
   cluster, you must provide a `--kind` flag. The `--kind` flag supports tab auto-completion. Run:

   ```console
   tanzu workload generate tanzu-java-web-app --kind appbuildv1s.supplychains.tanzu.vmware.com > workload.yaml
   ```

1. Add the necessary values in the `workload.yaml` file for each required entry. For example:

    ```yaml
    apiVersion: supplychains.tanzu.vmware.com/v1alpha1
    kind: AppBuildV1
    metadata:
      name: tanzu-java-web-app
    spec:
      gitOps:
        baseBranch: "main"
        subPath: "packages"
        url: GITOPS-REPO-PATH
      registry:
        repository: REPOSITORY-PATH
        server: REGISTRY-SERVER
      source:
        git:
          branch: "main"
          url: "https://github.com/vmware-tanzu/application-accelerator-samples.git"
        subPath: "tanzu-java-web-app"
      carvel:
        packageName: "tanzu-java-web-app"
        packageDomain: "tanzu.vmware.com"
    ```

   > **Caution** The beta version of the Tanzu Supply Chain does not support platform engineer-level
   > overrides and defaults. Therefore, the `Workload` generate command also shows the entries that
   > a platform engineer is supposed to set, such as the registry details.
   >
   > When the overrides feature is available, a platform engineer can set platform-level values, such
   > as the registry details. These entries are not part of the `generate` command output because that
   > is something a platform engineer does not want a developer to override.
   >
   > This causes a much smaller `Workload` specification that only has values that a developer can
   > provide for the `SupplyChain`. This helps to keep the platform engineering role and the developer
   > role separate.

1. Use the `tanzu workload apply` command to apply your `AppBuildV1` workload to the cluster. If
   your workload YAML file is not named `workload.yaml`, use the `-f` flag to point to it.
   For this tutorial, use the `dev` namespace. Run:

   ```console
   tanzu workload apply
   ```

   Example output:

   ```console
   🔎 Creating workload:
         1 + |---
         2 + |apiVersion: supplychains.tanzu.vmware.com/v1alpha1
         3 + |kind: AppBuildV1
         4 + |metadata:
         5 + |  name: tanzu-java-web-app
         6 + |  namespace: dev
         7 + |spec:
         8 + |  carvel:
         9 + |    packageDomain: tanzu.vmware.com
       10 + |    packageName: tanzu-java-web-app
       11 + |  gitOps:
       12 + |    baseBranch: main
       13 + |    subPath: packages
       14 + |    url: GITOPS-REPO-PATH
       15 + |  registry:
       16 + |    repository: REPOSITORY-PATH
       17 + |    server: REGISTRY-SERVER
       18 + |  source:
       19 + |    git:
       20 + |      branch: main
       21 + |      url: https://github.com/vmware-tanzu/application-accelerator-samples.git
       22 + |    subPath: tanzu-java-web-app
   Create workload tanzu-java-web-app from workload.yaml? [yN]: y
   ✓ Successfully created workload tanzu-java-web-app
   ```

1. See all the workloads of each kind running in your namespace by running:

   ```console
   tanzu workload list
   ```

   Example output:

   ```console
   Listing workloads from the dev namespace

     NAME                KIND                                       VERSION   AGE
     tanzu-java-web-app  appbuildv1s.supplychains.tanzu.vmware.com  v1alpha1  30m

   🔎 To see more details about a workload, use 'tanzu workload get workload-name --kind workload-kind'
   ```

1. See all the workloads running in all namespaces by running:

   ```console
   tanzu workload list -A
   ```

1. See how the `tanzu-java-web-app` workload in the workload list is progressing by running:

   ```console
   tanzu workload get tanzu-java-web-app
   ```

   Example output:

   ```console
   📡 Overview
      name:       tanzu-java-web-app
      kind:       appbuildv1s.supplychains.tanzu.vmware.com/tanzu-java-web-app
      namespace:  dev
      age:        15m

   🏃 Runs:
      ID                            STATUS   DURATION  AGE
      tanzu-java-web-app-run-454m5  Running  2m        2m

   🔎 To view a run information, use 'tanzu workload run get run-id'
   ```

   From the output, you see that a `WorkloadRun` named `tanzu-java-web-app-run-454m5` was created
   when you applied the `AppBuildV1` workload and it is in the `Running` state. There are multiple
   causes for why a new `WorkloadRun` is created for your `Workload`, but few are developer-triggered.

   Updates to your `Workload`, such as changing the values in `workload.yaml` and reapplying them to
   the cluster, create a new `WorkloadRun`. Platform engineering activities such as updating the
   Buildpack builder images can also cause your `Workload` to rebuild, creating a new `WorkloadRun`
   with newer base images.

   Other activities, such as pushing new commits to the source-code repository referenced by the
   `Workload`, can also cause a new run of the `Workload` to build the latest source from your Git
   repository.

1. Use the `tanzu workload run get` command to see more information about which stages your workload
   is going through. Use the optional `--show-details` flag for more detailed output. Run:

   ```console
   tanzu workload run get tanzu-java-web-app-run-454m5 --show-details
   ```

   Example output:

   ```console
   📡 Overview
      name:        tanzu-java-web-app
      kind:        appbuildv1s.supplychains.tanzu.vmware.com/tanzu-java-web-app
      run id:      appbuildv1runs.supplychains.tanzu.vmware.com/tanzu-java-web-app-run-454m5
      status:      Running
      namespace:   dev
      age:         14m

   📓 Spec
         1 + |---
         2 + |apiVersion: supplychains.tanzu.vmware.com/v1alpha1
         3 + |kind: AppBuildV1
         4 + |metadata:
         5 + |  name: tanzu-java-web-app
         6 + |  namespace: dev
         7 + |spec:
         8 + |  carvel:
         9 + |    packageDomain: tanzu.vmware.com
       10 + |    packageName: tanzu-java-web-app
       11 + |  gitOps:
       12 + |    baseBranch: main
       13 + |    subPath: packages
       14 + |    url: GITOPS-REPO-PATH
       15 + |  registry:
       16 + |    repository: REPOSITORY-PATH
       17 + |    server: REGISTRY-SERVER
       18 + |  source:
       19 + |    git:
       20 + |      branch: main
       21 + |      url: https://github.com/vmware-tanzu/application-accelerator-samples.git
       22 + |    subPath: tanzu-java-web-app

   🏃 Stages
      ├─ source-git-provider
      │  ├─ 📋 check-source - Success
      │  │  ├─ Duration: 31s
      │  │  └─ Results
      │  │     ├─ message: using git-branch: main
      │  │     ├─ sha: e4e23867bcffcbf7a165e2fefe3c48dc28b076d6
      │  │     └─ url: https://github.com/vmware-tanzu/application-accelerator-samples.git
      │  └─ 📋 pipeline - Success
      │     ├─ Duration: 1m28s
      │     └─ Results
      │        ├─ url: IMAGE-URL
      │        └─ digest: IMAGE-SHA
      ├─ buildpack-build
      │  ├─ 📋 check-builders - Success
      │  │  ├─ Duration: 26s
      │  │  └─ Results
      │  │     ├─ builder-image: BUILDER-IMAGE-USED
      │  │     ├─ message: Builders resolved
      │  │     └─ run-image: RUN-IMAGE-USED
      │  └─ 📋 pipeline - Success
      │     ├─ Duration: 2m59s
      │     └─ Results
      │        ├─ url: IMAGE-URL
      │        └─ digest: IMAGE-SHA
      ├─ conventions
      │  └─ 📋 pipeline - Success
      │     ├─ Duration: 33s
      │     └─ Results
      │        ├─ url: IMAGE-URL
      │        └─ digest: IMAGE-SHA
      ├─ app-config-server
      │  └─ 📋 pipeline - Running
      │     └─ Duration: 22.37089s
      ├─ carvel-package
      │  └─ 📋 pipeline - Not Started
      └─ git-writer-pr
         └─ 📋 pipeline - Not Started
   ```

The output shows you:

- An overview of your workload, such as `name`, `kind`, and `namespace`
- The last 2 successful `WorkloadRuns`
- The last failed `WorkloadRun`
- All running `WorkloadRuns`
- The error section from the last failed `WorkloadRun`

When your `WorkloadRun` has gone through the Supply Chain, the output of the `Workload` and
`WorkloadRun` is as follows:

Workload Run Output
: **tanzu workload run get tanzu-java-web-app-run-454m5 --show-details**

    ```console
    📡 Overview
      name:        tanzu-java-web-app
      kind:        appbuildv1s.supplychains.tanzu.vmware.com/tanzu-java-web-app
      run id:      appbuildv1runs.supplychains.tanzu.vmware.com/tanzu-java-web-app-run-454m5
      status:      Succeeded
      namespace:   dev
      age:         68m

    📓 Spec
      1 + |---
      2 + |apiVersion: supplychains.tanzu.vmware.com/v1alpha1
      3 + |kind: AppBuildV1
      4 + |metadata:
      5 + |  name: tanzu-java-web-app
      6 + |  namespace: dev
      7 + |spec:
      8 + |  carvel:
      9 + |    packageDomain: tanzu.vmware.com
      10 + |    packageName: tanzu-java-web-app
      11 + |  gitOps:
      12 + |    baseBranch: main
      13 + |    subPath: packages
      14 + |    url: GITOPS-REPO-PATH
      15 + |  registry:
      16 + |    repository: REPOSITORY-PATH
      17 + |    server: REGISTRY-SERVER
      18 + |  source:
      19 + |    git:
      20 + |      branch: main
      21 + |      url: https://github.com/vmware-tanzu/application-accelerator-samples.git
      22 + |    subPath: tanzu-java-web-app

    🏃 Stages
      ├─ source-git-provider
      │  ├─ 📋 check-source - Success
      │  │  ├─ Duration: 31s
      │  │  └─ Results
      │  │     ├─ message: using git-branch: main
      │  │     ├─ sha: e4e23867bcffcbf7a165e2fefe3c48dc28b076d6
      │  │     └─ url: https://github.com/vmware-tanzu/application-accelerator-samples.git
      │  └─ 📋 pipeline - Success
      │     ├─ Duration: 1m28s
      │     └─ Results
      │        ├─ url: IMAGE-URL
      │        └─ digest: IMAGE-SHA
      ├─ buildpack-build
      │  ├─ 📋 check-builders - Success
      │  │  ├─ Duration: 26s
      │  │  └─ Results
      │  │     ├─ builder-image: BUILDER-IMAGE-USED
      │  │     ├─ message: Builders resolved
      │  │     └─ run-image: RUN-IMAGE-USED
      │  └─ 📋 pipeline - Success
      │     ├─ Duration: 2m59s
      │     └─ Results
      │        ├─ url: IMAGE-URL
      │        └─ digest: IMAGE-SHA
      ├─ conventions
      │  └─ 📋 pipeline - Success
      │     ├─ Duration: 33s
      │     └─ Results
      │        ├─ url: IMAGE-URL
      │        └─ digest: IMAGE-SHA
      ├─ app-config-server
      │  └─ 📋 pipeline - Success
      │     ├─ Duration: 1m12s
      │     └─ Results
      │        ├─ url: IMAGE-URL
      │        ├─ digest: IMAGE-SHA
      │        ├─ url-overlay: IMAGE-URL
      │        └─ digest-overlay: IMAGE-SHA
      ├─ carvel-package
      │  └─ 📋 pipeline - Success
      │     ├─ Duration: 49s
      │     └─ Results
      │        ├─ url: IMAGE-URL
      │        └─ digest: IMAGE-SHA
      └─ git-writer-pr
      └─ 📋 pipeline - Success
          ├─ Duration: 34s
          └─ Results
              ├─ url: PULL-REQUEST-URL-TO-GITOPS-REPO
              └─ digest: IMAGE-SHA
    ```

Workload Get Output
: **tanzu workload get tanzu-java-web-app**

    ```console
    📡 Overview
    name:       tanzu-java-web-app
    kind:       appbuildv1s.supplychains.tanzu.vmware.com/tanzu-java-web-app
    namespace:  dev
    age:        74m

    🏃 Runs:
    ID                            STATUS     DURATION  AGE
    tanzu-java-web-app-run-454m5  Succeeded  16m       72m

    🔎 To view a run information, use 'tanzu workload run get run-id'
    ```

Based on the description of the `AppBuildV1` kind from the `tanzu workload kind list` command, the
Supply Chain pulls the source code from the Git repository, builds it by using buildpacks, and then
packages the output as a Carvel package. That output is then shipped to the GitOps repository that
the platform engineer configures.

In your Supply Chain, when the `WorkloadRun` succeeds you can see the URL for the pull request to
the GitOps repository in the `tanzu workload run get --show-details` output in the `gitops-pr` stage
results.

You have now used Tanzu Supply Chain to deploy your first workload.

## <a id="next-steps"></a> Next Steps

See these [How-to guides](./../how-to/about.hbs.md) for developers to learn more about Tanzu Supply
Chain.