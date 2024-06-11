# Test accelerators in Application Accelerator

This topic tells you how to test an updated accelerator or fragment that is not registered in your
Tanzu Application Platform (commonly known as TAP) cluster.

## <a id="accel-rapid-iteration"></a> Generate a project from local sources

When you are authoring your accelerator, you can test it before committing any changes.

With the `tanzu accelerator generate-from-local` command, you can run your accelerator (or
fragment), including any changes you have locally, specify a set of options and view the
generated project.

You can run the accelerator using the components on your Tanzu Application Platform cluster,
without impacting the state of the Tanzu Application Platform cluster.

To do so, ensure that you have the following prerequisites:

- The Tanzu CLI is installed, with the Application Accelerator plug-in. For details about installing
  the Tanzu CLI and plug-ins, see [Tanzu CLI](../../cli-plugins/tanzu-cli.hbs.md).
- The server URL is pointing to the Tanzu Application Platform cluster you want to test with. For
  details about setting the server URL, see
  [Application Accelerator CLI plug-in overview](../../cli-plugins/accelerator/overview.hbs.md).

For example, to use the accelerator that is located at the path `workspace/java-rest`:

```console
tanzu accelerator generate-from-local --accelerator-path java-rest=workspace/java-rest \
--fragment-names tap-workload,java-version --options '{"projectName":"test"}' --output-dir \
generated-project
```

This generates the project in the local directory `generated-project`, using the accelerator located
at `workspace/java-rest`, the fragments `tap-workload` and `java-version` which are assumed to be
already registered in the Tanzu Application Platform cluster and the option `projectName` set to
`test`.

For example, to use the fragment named `java-version` that is located at the path `workspace/version`:

```console
tanzu accelerator generate-from-local --accelerator-name java-rest --fragment-paths \
java-version=workspace/version --fragment-names tap-workload --options '{"projectName":"test"}' \
--output-dir generated-project
```

This generates the project in the local directory `generated-project`,
using the accelerator `java-rest` and the fragment `tap-workload` which are assumed to be
already registered in the Tanzu Application Platform cluster, the fragment named `java-version`
located at `workspace/version`, and the option `projectName` set to `test`.

For more information about the `generate-from-local` command, see the
[Tanzu CLI Command Reference](https://docs.vmware.com/en/VMware-Tanzu-CLI/{{vars.tanzu-cli.url_version }}/tanzu-cli/command-ref.html)
documentation.

No changes are made to the Tanzu Application Platform cluster that is provided with the server URL.
No new accelerators/fragments are registered or modified. A Tanzu Application Platform cluster is
required to ensure that there is consistency between the version that is used for testing and the
version that is used when the accelerator is registered. Furthermore, it allows using registered
fragments and accelerators as dependencies for the local accelerator/fragment.

## <a id="creating-accel-ci-cd-pl"></a> CI/CD Pipeline

As you iterate on an accelerator, you can have some automated assertions run before any changes to
the accelerator are accepted.

The process for generating a project from the committed source files is the same as described earlier.

When the generated project is available, run various assertions on it by running:

```console
cd generated-project
test -f build.gradle
./gradlew test
```

If you have multiple assertions, you can use a predefined script by running:

```console
cd generated-project
../assertions/validate-generate-project.sh
```

You can generate multiple projects from the same accelerator, providing different options for each
accelerator and running different assertions on each generated project.

### <a id="tanzu-cli-in-ci-cd"></a> Add the Tanzu CLI to a CI/CD pipeline

If the Tanzu CLI is already available in your CI/CD pipeline, skip this section.

To add the Tanzu CLI to a CI/CD pipeline:

1. Follow the steps in
   [Installing the Tanzu CLI](https://github.com/vmware-tanzu/tanzu-cli/blob/main/docs/quickstart/install.md).
   Install the binary file that matches the operating system and architecture that you use in your CI.

1. From your CI script, install the `accelerator` plug-in by running these commands:

   ```console
   tanzu config eula accept
   tanzu ceip-participation set false
   tanzu init
   tanzu plugin install accelerator --target kubernetes
   ```
