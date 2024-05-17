# Install the SonarQube Scan component (alpha)

This topic tells you how to install the SonarQube Scan component and customize the image to run the
SonarQube scan on.

> **Note** The SonarQube Scan component is in the alpha stage and only supports Maven projects at
> this time.

If you used the `authoring` profile, the SonarQube Scan component is installed by default. For more
information, see
[Install Tanzu Supply Chain with the authoring profile](../../../supply-chain/platform-engineering/how-to/installing-supply-chain/install-authoring-profile.hbs.md#tsc-packages).

This section describes how to install the SonarQube Scan component by itself, without using
the `authoring` profile.

1. List version information for the SonarQube Scan component package by running:

   ```console
   tanzu package available list sonarqube.component.apps.tanzu.vmware.com --namespace tap-install
   ```

   For example:

   ```console
   $ tanzu package available list sonarqube.component.apps.tanzu.vmware.com --namespace tap-install

   NAME                                       VERSION      RELEASED-AT
   sonarqube.component.apps.tanzu.vmware.com  0.1.0        2024-05-01 17:13:39 -0400 EDT
   ```

1. Install the SonarQube Scan component package by running:

   ```console
   tanzu package install sonarqube-scan-component -p sonarqube.component.apps.tanzu.vmware.com \
       --version SONARQUBE-COMPONENT-VERSION \
       --namespace tap-install
   ```

   Where `SONARQUBE-COMPONENT-VERSION` is the version that you want.
