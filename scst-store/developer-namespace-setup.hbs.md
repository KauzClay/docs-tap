# Developer namespace setup for Supply Chain Security Tools - Store

This topic tells how you to set up your developer namespace for Supply Chain Security Tools (SCST) -
Store.

## Overview

After you finish installing Tanzu Application Platform, you are ready to configure the developer
namespace. When you configure a developer namespace, you must export the SCST - Store certificate
authority (CA) certificate and authentication token to the namespace. This enables SCST - Scan to
find the credentials to send scan results to SCST - Store.

There are two ways to deploy Tanzu Application Platform:

- The single-cluster method, which uses the Tanzu Application Platform values file
- THe multicluster method, which uses `SecretExport`

## Single cluster - using the Tanzu Application Platform values file

To use `tap-values.yaml` to deploy Tanzu Application Platform, if using a single cluster:

1. When deploying the Tanzu Application Platform Full or Build profile, edit `tap-values.yaml` as
   follows:

    ```yaml
    metadata_store:
      ns_for_export_app_cert: "DEV-NAMESPACE"
    ```

   Where `DEV-NAMESPACE` is the name of the developer namespace.

   The `ns_for_export_app_cert` supports only one namespace at a time. If you have multiple
   namespaces, you can replace this value with a `"*"`. For example:

    ```yaml
    metadata_store:
      ns_for_export_app_cert: "*"
    ```

> **Caution** The use `"*"` entails exporting the CA to all namespaces. Consider whether this
> increased visibility is a risk.

1. Update Tanzu Application Platform to apply the changes by running:

   ```console
   tanzu package installed update tap -f tap-values.yaml -n tap-install
   ```

## Multicluster - using `SecretExport`

In a multicluster deployment, follow the steps in
[Multicluster setup for SCST - Store](multicluster-setup.hbs.md), which describe how to create
secrets and export secrets to developer namespaces.

## Next steps

If you arrived in this topic from
[Out of the Box Supply Chain with Testing and Scanning for Supply Chain Choreographer](../scc/ootb-supply-chain-testing-scanning.hbs.md#storing-scan-results),
return to that topic and continue with the instructions.