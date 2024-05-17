# Setup the SonarQube Supply Chain Component

This topic describes how to install the SonarQube Supply Chain Component, and customize the image to run the SonarQube scan on.

## <a id="install-sonarqube-sc"></a> Install SonarQube Supply Chain Component

With the `authoring` profile, the SonarQube scan component should be installed by default. See instructions on the [authoring profile](../../supply-chain/platform-engineering/how-to/installing-supply-chain/install-authoring-profile.hbs.md#tsc-packages).

This section describes how to install the SonarQube Supply Chain Component by itself, without using the `authoring` profile.

1. List version information for the SonarQube Supply Chain Component package. Run:

    ```console
    tanzu package available list sonarqube.component.apps.tanzu.vmware.com --namespace tap-install
    ```

    For example:

    ```console
    $ tanzu package available list sonarqube.component.apps.tanzu.vmware.com --namespace tap-install

    NAME                                       VERSION      RELEASED-AT
    sonarqube.component.apps.tanzu.vmware.com  0.1.0        2024-05-01 17:13:39 -0400 EDT
    ```

2. Install the SonarQube Supply Chain Component package.

    ```console
    tanzu package install sonarqube-scan-component -p sonarqube.component.apps.tanzu.vmware.com \
        --version SONARQUBE-COMPONENT-VERSION \
        --namespace tap-install
    ```
