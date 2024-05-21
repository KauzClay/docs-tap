# Install the Tanzu Supply Chain CLI plug-ins

This topic tells you how to install the Tanzu Supply Chain and Tanzu Workload CLI plug-ins.

{{> 'partials/supply-chain/beta-banner' }}

To work with Tanzu Supply Chain, there are two Tanzu CLI plug-ins that you must install:

- The Tanzu Supply Chain CLI plug-in, with which you author and manage `SupplyChain` resources and
  `Component` resources
- The Tanzu Workload CLI plug-in, which developers use to discover and use the `Workload` resources
  that you authored with the Tanzu Supply Chain CLI plug-in

> **Important** The Tanzu Supply Chain beta release does not include the installation of the Tanzu
> Workload and Supply Chain CLI plug-ins as part of the plug-in group.

## Prerequisites

Ensure that you installed or updated the core Tanzu CLI. For more information, see
[Install Tanzu CLI](../../../install-tanzu-cli.hbs.md#install-cli).

## Install Tanzu Supply Chain CLI plug-ins

To install Tanzu Supply Chain CLI plug-ins:

1. Install the Tanzu CLI plug-ins by running:

   ```console
   tanzu plugin install supplychain
   tanzu plugin install workload
   ```

2. Verify that the plug-ins are installed correctly by running:

   ```console
   tanzu supplychain version
   tanzu workload version
   ```

## Uninstall Tanzu Supply Chain CLI plug-ins

To uninstall Tanzu Supply Chain CLI plug-ins, run:

```console
tanzu plugin delete supplychain
tanzu plugin delete workload
```
