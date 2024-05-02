# Install Enterprise Config Server

This topic tells you how to install Enterprise Config Server from the Tanzu Application Platform
(commonly known as TAP) package repository.

## <a id='prepare'></a> Prepare

Before installing Config Server, you must:

- Fulfill all [prerequisites for installing Tanzu Application Platform](../prerequisites.hbs.md)
- [Install cert-manager](../cert-manager/install.hbs.md)

## <a id='install'></a> Install

To install Config Server on a compliant Kubernetes cluster:

1. List version information for the package by running:

   ```console
   tanzu package available list config-server.spring.tanzu.vmware.com --namespace tap-install
   ```

   For example:

   ```console
   $ tanzu package available list config-server.spring.tanzu.vmware.com --namespace tap-install

     NAME                                   VERSION  RELEASED-AT
     config-server.spring.tanzu.vmware.com  1.0.0    2024-04-29 11:33:42 -0500 CDT
   ```

1. Install the package by running:

   ```console
   tanzu package install config-server \
   --package config-server.spring.tanzu.vmware.com \
   --version VERSION -n tap-install
   ```

   Where `VERSION` is the version you want, such as `1.0.0`.

   For example:

   ```console
   $ tanzu package install config-server \
     --package config-server.spring.tanzu.vmware.com \
     --version 1.0.0 -n tap-install
    Installing package 'config-server.spring.tanzu.vmware.com'
    Getting package metadata for 'config-server.spring.tanzu.vmware.com'
    Creating service account 'config-server-tap-install-sa'
    Creating cluster admin role 'config-server-tap-install-cluster-role'
    Creating cluster role binding 'config-server-tap-install-cluster-rolebinding'
    Creating package resource
    Waiting for 'PackageInstall' reconciliation for 'config-server'
    'PackageInstall' resource install status: Reconciling
    'PackageInstall' resource install status: ReconcileSucceeded
    'PackageInstall' resource successfully reconciled

    Added installed package 'config-server'
   ```

   > **Note** Because there are no customization options at this time, you need not include a
   > `--values-file` option.

1. Verify that you installed the package by running:

   ```console
   tanzu package installed get config-server -n tap-install
   ```

   For example:

   ```console
   $ tanzu package installed get config-server -n tap-install

   NAMESPACE:               tap-install
   NAME:                    config-server
   PACKAGE-NAME:            config-server.spring.tanzu.vmware.com
   PACKAGE-VERSION:         1.0.0
   STATUS:                  Reconcile succeeded
   ```
