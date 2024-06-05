# Use a local Application Accelerator engine server

This topic tells you how to run a local Application Accelerator engine server that you can use for
testing accelerators that you are authoring.

## <a id="overview"></a> About running a local engine server

When you are authoring your accelerator, it might be convenient to generate a project based on the local
files. This enables you to verify that the accelerator provides the defined options and generates
the correct set of files.

With the local engine server, you can serve your accelerators with their fragments on `localhost`.
This includes any changes you have locally. You can generate new projects based on these local files
using the Visual Studio Code (VS Code) Tanzu App Accelerator extension,
IntelliJ Tanzu Application Accelerator plug-in, or the Tanzu CLI Accelerator plug-in.
After you are satisfied with the new or modified accelerators and fragments, you can commit them to
a Git repository and then publish them to a cluster to give others access.

## <a id="install-local-engine-server"></a>Install the local engine server

To install the local engine server:

1. Go to the [Broadcom Support Portal](https://support.broadcom.com/group/ecx/productdownloads?subfamily=Tanzu+Application+Platform+(TAP)), expand the **VMware Tanzu Application Platform**
dropdown, and click the {{ vars.tap_version }} release.

1. If you have not done so already, select the **I agree to Terms and Conditions** check box.

1. From the list of resources, download the acc-engine binary. Choose from **acc-engine-linux**,
   **acc-engine-macos-aarch64**, **acc-engine-macos-amd64**, or **acc-engine-windows** depending on your
   operating system and architecture.

    The ZIP file contains the local engine server and a Java runtime for
    running the server.

1. Extract the ZIP file to a local directory by using the `unzip` command or any other extraction tool.

1. For macOS users, you must give permission to open an app from an unidentified developer:

    1. In the Finder on your Mac, locate the directory where you extracted the downloaded ZIP file
       and expand the `acc-engine` directory.

    1. Control-open the following files:

        - Control-click `acc-engine/app/bin/ytt` and then click **Open**. This runs it in a terminal
        that you can close.

        - Control-click `acc-engine/app/bin/java` and then click **Open**. This runs it in a terminal
        that you can close.

        The app files you control-opened are saved as exceptions to your security settings.
        You can now run them without getting a verification message.

        > **Note** VMware plans to have these artifacts signed using an Apple developer account
        > to avoid these extra steps.

1. Open a terminal window and change directory to `acc-engine` located inside the directory where
   you extracted the ZIP file.

1. Set an environment variable named `ACC_LOCAL_FILES` that points to a directory that contains the
   fragments and accelerators you want to use with the local engine server.
   There must be a directory named `accelerators` and one named `fragments`.
   Under these directories, you can provide your local accelerators and fragments. For example:

    ```console
      workspace
      ├── accelerators
      │   └── hello-world
      │       ├── accelerator.yaml
      │       ├── accelerator.axl
      │       ├── ...
      └── fragments
          ├── build-wrapper-maven
          │   ├── accelerator.yaml
          │   ├── accelerator.axl
          │   ├── ...
          ├── java-version
          │   ├── accelerator.yaml
          │   ├── accelerator.axl
          │   ├── ...
    ```

    For more examples, see [application-accelerator-samples](https://github.com/vmware-tanzu/application-accelerator-samples) in GitHub.

    - **For macOS and Linux:** Set the environment variable, for example:

        ```console
        $ export ACC_LOCAL_FILES="$HOME/workspace"
        ```

    - **For Windows Powershell:** Set the environment variable, for example:

        ```console
        $ $Env:ACC_LOCAL_FILES="$HOME\workspace"
        ```

## <a id="use-local-engine-server"></a>Use the local engine server to generate projects

To use the local engine server:

1. Start the local engine server by running the `engine` script from the terminal:

    ```console
    ./engine
    ```

1. The latest versions of the VS Code Tanzu App Accelerator extension, Tanzu Application Accelerator plug-in for IntelliJ,
   and the Tanzu CLI Accelerator plug-in have settings to use the local engine server instead of the regular cluster endpoints.

    - **For the VS Code Tanzu App Accelerator extension:**

      Under **Settings** > **Extensions** > **Tanzu Application Accelerator**, select the
      **Use Local Server instead of Developer Portal** check box.

      The extension can then show accelerators from the local engine server that you started.
      Use them in the same way that you use accelerators loaded from Tanzu Developer Portal.
      For more information, see [Use the extension](../vscode.hbs.md#using-the-extension).

    - **For the Tanzu Application Accelerator plug-in for IntelliJ:**

      Under **IntelliJ IDEA** > **Preferences** > **Tools** > **Tanzu Application Accelerator**,
      select the **Use Local Server instead of Developer Portal** check box.

      The plug-in can then show accelerators from the local engine server that you started.
      Use them in the same way that you use accelerators loaded from Tanzu Developer Portal.
      For more information, see [Use the plug-in](../intellij.hbs.md#intellij-using-the-plugin).

    - **For the Tanzu CLI Accelerator plug-in:**

      For the `list`, `get`, and `generate` commands, use the `--local-server` flag instead of `--server-url`.

## <a id="test-script"></a> Create a test suite for your accelerator

To create a test suite to test your accelerator using the local engine server:

1. Create an `options.json` file using **Export Options** in the Tanzu App Accelerator IDE extensions
   to export the option values selected for the accelerator.

    - For instructions for VS Code, see [Export accelerator configuration options (VS Code)](../vscode.hbs.md#export-options).
    - For instructions for IntelliJ, see [Export accelerator configuration options (IntelliJ)](../intellij.hbs.md#export-options).

1. Create an `assertions.sh` file that contains a BASH script that checks the content of the generated project.

1. Organize your `options.json` files and `assertions.sh` scripts into a directory structure for each
   accelerator to test.

    The following is an example directory structure for the `spring-cloud-serverless` accelerator:

    ```console
    .
    ├── spring-cloud-serverless-k8s
    │   ├── assertions.sh
    │   └── options.json
    └── spring-cloud-serverless-tap
        ├── assertions.sh
        └── options.json
    ```

For an example test suite, see [application-accelerator-samples](https://github.com/vmware-tanzu/application-accelerator-samples/tree/main/local-test-suite-example)
in GitHub.
