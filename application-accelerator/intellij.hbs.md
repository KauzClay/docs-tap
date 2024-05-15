# Use the Application Accelerator IntelliJ plug-in

This topic tells you about the Application Accelerator IntelliJ plug-in. The plug-in is used to
explore and generate projects from the defined accelerators in Tanzu Application Platform
(commonly called TAP) using IntelliJ.

## <a id="dependencies"></a> Dependencies

The plug-in must have access to the Tanzu Developer Portal URL.
For information about how to retrieve the Tanzu Developer Portal URL, see
[Retrieving the URL for the Tanzu Developer Portal](#fqdn-tap-gui-url) later in this topic.

## <a id="intellij-install"></a> Install the plug-in

The VMware Tanzu Application Accelerator plug-in for IntelliJ is available from the
[JetBrains Marketplace](https://plugins.jetbrains.com/plugin/23645-tanzu-application-accelerator).

To install the plug-in from the JetBrains Marketplace:

1. Open IntelliJ.
2. Open the command palette, enter `Plugins`, and click **Plugins**.
3. Select the **Marketplace** tab in the **Plugins Settings** dialog box.
4. In the search box, enter `Tanzu`.
5. Click **Tanzu Application Accelerator** then click **Install**.

## <a id="update"></a> Update the plug-in

To update to a later version, repeat the steps in [Install the plug-in](#intellij-install).
You do not need to uninstall your current version.

## <a id="intellij-conf-plugin"></a> Configure the plug-in

Before using the plug-in, you must enter the Tanzu Developer Portal URL in the IntelliJ Preferences:

1. Go to the IntelliJ menu, select **IntelliJ IDEA > Preferences** > **Tools** > **Tanzu Application Accelerator**.

2. Add the Tanzu Developer Portal URL. For example, `https://tap-gui.myclusterdomain.myorg.com`.
If you have access to the Tanzu Application Platform cluster that is running the
Tanzu Developer Portal, run the following command to determine the fully-qualified domain name:

    ```console
    kubectl get httpproxy tap-gui -n tap-gui
    ```

    ![Tanzu Application Accelerator preferences.](../images/app-accelerator/intellij/app-accelerators-intellij-preferences.png)

3. Click **Apply** and **OK**.

## <a id="intellij-using-the-plugin"></a> Use the plug-in

To use the IntelliJ plug-in:

1. To explore the defined accelerators, select **New Project**, then select **Tanzu Application Accelerator**.

   ![The IntelliJ UI with the New Project button highlighted.](../images/app-accelerator/intellij/app-accelerators-intellij-new-project.png)

   ![Tanzu Application Accelerator New Project wizard is open.](../images/app-accelerator/intellij/app-accelerators-intellij-accelerator-list.png)

2. Choose one of the defined accelerators and configure the options.

   ![Options page is open.](../images/app-accelerator/intellij/app-accelerators-intellij-options.png)

3. Click `Next` to go to the optional step `create git repository`.

   ![Review page is open.](../images/app-accelerator/intellij/app-accelerators-intellij-review.png)

4. Fill the fields required to create the Git repository, a personal access token from the Git
provider is required, this will be stored in a secured location for future use, click `Next` to
go to the review step.

   ![Git repository creation.](../images/app-accelerator/intellij/app-accelerators-intellij-git-repo-creation.png)

   > Note: This is an optional step, the values can be left blank if a repository isn't required.

5. Click `Next` to download the project. If the project is downloaded successfully, the `Create` button is enabled and you can now create and open the project.

## <a id="export-options-intellij"></a> Export accelerator configuration options

For faster iteration while writing accelerators, you can export the accelerator options in the
**Review and Generate Step** to a JSON file. You can use this file to generate your project again from the CLI.

To export your options:

1. Select the accelerator that you want to use in the new project wizard described above, and configure it in the following **Options Step**

1. In the **Preview and Generate Step**, towards the bottom there is a field called **Export Options**.

1. Enter a file name and click the **Export** button to select a location to save the file.

  ![Export Options](../images/app-accelerator/intellij/export-options-intellij.png)

## <a id="fqdn-tap-gui-url"></a> Retrieving the URL for the Tanzu Developer Portal

If you have access to the Tanzu Application Platform cluster that is running the Tanzu Application
Platform GUI, run the following command to determine the fully-qualified domain name:

```console
kubectl get httpproxy tap-gui -n tap-gui
```

The result is similar to:

```console
NAME      FQDN                                      TLS SECRET     STATUS   STATUS DESCRIPTION
tap-gui   tap-gui.tap.tapdemo.myorg.com             tap-gui-cert   valid    Valid HTTPProxy
```

## <a id="dl-ins-ss-certs"></a>Download and Install Self-Signed Certificates

To enable communication between the Application Accelerator plug-in and a Tanzu Application Platform
GUI instance that is secured using TLS, you must download and install the certificates locally.

### Prerequisites

[yq](https://github.com/mikefarah/yq) is required to process the YAML output.

### Procedure

1. Find the name of the Tanzu Developer Portal certificate. The name of the certificate
might look different to the following example.

    ```console
    kubectl get secret -n cert-manager
    ```

    ```console
    NAME                                           TYPE                             DATA   AGE
    canonical-registry-secret                      kubernetes.io/dockerconfigjson   1      18d
    cert-manager-webhook-ca                        Opaque                           3      18d
    postgres-operator-ca-certificate               kubernetes.io/tls                3      18d
    tanzu-sql-with-mysql-operator-ca-certificate   kubernetes.io/tls                3      18d
    tap-ingress-selfsigned-root-ca                 kubernetes.io/tls                3      18d <------- This is the certificate that is needed
    ```

2. Download the certificate:

    ```console
    kubectl get secret -n cert-manager tap-ingress-selfsigned-root-ca -o yaml | yq '.data."ca.crt"' | base64 -d > ca.crt
    ```

3. Install the certificate on your local system and restart any applications that use
the certificate. After restarting, the application uses the certificate
to communicate with the endpoints using TLS.

  macOS
  : Run:

  ```console
  sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain ca.crt
  ```

  For more information, see [Installing a root CA certificate in the trust store](https://ubuntu.com/server/docs/security-trust-store) in the Ubuntu documentation.

  Windows
  : Complete the following steps:

    1. Use Windows Explorer to navigate to the directory where the certificate was downloaded and select the certificate.
    2. In the Certificate window, click **Install Certificate...**.
    3. Change the **Store Location** from **Current User** to **Local Machine**. Click **Next**.
    4. Select **Place all certificates in the following store**, click **Browse**, and select **Trusted Root Certification Authorities**
    5. Click **Finish**.
    6. A pop-up window stating **The import was successful.** is displayed.

## <a id="uninstall"></a> Uninstall the plug-in

To uninstall the VMware Tanzu Application Accelerator plug-in for IntelliJ:

1. Open the **Preferences** pane and go to **Plugins**.
2. Select the extension, click the gear icon, and click **Uninstall**.
3. Restart IntelliJ.

## <a id="known-issues-intellij"></a>Known Issues

When creating a Java project using the Accelerator new project wizard in IntelliJ, it may sometimes not build correctly when first opened. 

This is particularly the case for Maven projects. When the newly created project is opened for the first time, a pop-up dialogue may appear in the bottom right side of IntelliJ asking to **Load Maven Project**.

  ![Load Maven Project](../images/app-accelerator/intellij/load-maven-project-intellij.png)

Click **Load Maven Project** and wait until the project builds. 

If this does not fix the issue, another possible workaround is to delete
the .idea folder and *.iml file, and the reopen the project.

  ![Delete IDEA files](../images/app-accelerator/intellij/delete-idea-files.png)




