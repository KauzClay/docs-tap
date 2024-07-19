# Install the Tanzu Workload CLI plug-in

This topic tells you how to install the Tanzu Workload CLI plug-in. Use this CLI plug-in to work
with the workloads provided by platform engineering.

{{> 'partials/supply-chain/beta-banner' }}

This plug-in enables you to see workload kinds, deploy workloads, and monitor the workloads. The
Tanzu Supply Chain beta release does not include the installation of the Tanzu Workload CLI plug-in
as part of the plug-in group.

The Tanzu Workload CLI plug-in provides commands that enable a developer to generate a `Workload`
manifest, apply it to a cluster, and delete it from a cluster.

## Prepare

Install the latest version of the Tanzu CLI. For more information, see
[Install Tanzu CLI](../../../install-tanzu-cli.hbs.md#install-cli).

## Install Tanzu Workload CLI plug-in

To install the Tanzu Workload CLI plug-in:

1. Install the plug-in by running:

   ```console
   tanzu plugin install workload
   ```

1. Verify that you installed the plug-in by running:

   ```console
   tanzu workload version
   ```

## Uninstall Tanzu Workload CLI plug-in

To uninstall the Tanzu Workload CLI plug-in, run:

```console
tanzu plugin delete workload
```