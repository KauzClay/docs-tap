# Overview of Tanzu CLI

This topic tells you about the Tanzu command-line interface (commonly known as Tanzu CLI).

## <a id="tanzu-cli"></a>Tanzu CLI

The Tanzu CLI is a command-line interface that connects you to Tanzu. For example, some functions of
Tanzu CLI include:

- Configure the Tanzu CLI itself
- Install and manage packages
- Run Tanzu services and components
- Create and manage application workloads

For more information about Tanzu CLI, see the
[VMware Tanzu CLI documentation](https://docs.vmware.com/en/VMware-Tanzu-CLI/index.html).

## <a id="itanzu-cli-architecture"></a>Tanzu CLI architecture

The Tanzu CLI has a pluggable architecture. Plug-ins extend the Tanzu CLI core with additional CLI
commands. Use the `vmware-tap/default:v{{ vars.tanzu-cli.plugin_group_version }}` plug-in group with Tanzu Application Platform. For more information,
see [Install Tanzu CLI plug-ins](../install-tanzu-cli.hbs.md).

The `vmware-tap/default:v{{ vars.tanzu-cli.plugin_group_version }}` plug-in group consists of the following plug-ins:

```console
NAME              DESCRIPTION                                                      TARGET      VERSION
  telemetry         configure cluster-wide settings for vmware tanzu telemetry       global      v1.1.0
  accelerator       Manage accelerators in a Kubernetes cluster                      kubernetes  v1.7.0
  apps              Applications on Kubernetes                                       kubernetes  v0.12.1
  build-service     plugin to interact with tanzu build service (tbs) crds           kubernetes  v1.0.0
  external-secrets  interacts with external-secrets.io resources                     kubernetes  v0.1.0
  package           Tanzu package management                                         kubernetes  v0.29.0
  secret            Tanzu secret management                                          kubernetes  v0.29.0
  services          Commands for working with service instances, classes and claims  kubernetes  v0.8.0
```

> **Note** The Telemetry, Package, and Secret plug-ins are not Tanzu Application Platform specific, they
are common to a number of other plug-in groups, for more information, see
 [Command Groups](https://docs.vmware.com/en/VMware-Tanzu-CLI/{{ vars.tanzu-cli.url_version }}/tanzu-cli/command-ref.html) in the
Tanzu CLI Product Documentation.

| Command Reference Documentation | Release Notes |
|:--------|:---------|
|[Accelerator command reference](https://docs.vmware.com/en/VMware-Tanzu-CLI/{{ vars.tanzu-cli.url_version }}/tanzu-cli/tanzu-accelerator.html)|[Accelerator release notes](https://docs.vmware.com/en/VMware-Tanzu-CLI/{{ vars.tanzu-cli.url_version }}/tanzu-cli/release-notes-accelerator.html)|
|[Apps command reference](https://docs.vmware.com/en/VMware-Tanzu-CLI/{{ vars.tanzu-cli.url_version }}/tanzu-cli/tanzu-apps.html)|[Apps release notes](https://docs.vmware.com/en/VMware-Tanzu-CLI/{{ vars.tanzu-cli.url_version }}/tanzu-cli/release-notes-apps.html)|
|[Build Service command reference](https://docs.vmware.com/en/VMware-Tanzu-CLI/{{ vars.tanzu-cli.url_version }}/tanzu-cli/tanzu-build-service.html)|[Build Service release notes](https://docs.vmware.com/en/VMware-Tanzu-CLI/{{ vars.tanzu-cli.url_version }}/tanzu-cli/release-notes-build-service.html)|
|[External Secrets command reference](https://docs.vmware.com/en/VMware-Tanzu-CLI/{{ vars.tanzu-cli.url_version }}/tanzu-cli/tanzu-external-secrets.html)|[External Secrets release notes](https://docs.vmware.com/en/VMware-Tanzu-CLI/{{ vars.tanzu-cli.url_version }}/tanzu-cli/release-notes-external-secrets.html)|
|[Services command reference](https://docs.vmware.com/en/VMware-Tanzu-CLI/{{ vars.tanzu-cli.url_version }}/tanzu-cli/tanzu-services.html)|[Services release notes](https://docs.vmware.com/en/VMware-Tanzu-CLI/{{ vars.tanzu-cli.url_version }}/tanzu-cli/release-notes-services.html)|

## <a id="tanzu-cli-install"></a>Tanzu CLI installation

To install the Tanzu CLI, see [Install Tanzu CLI](../install-tanzu-cli.hbs.md#cli-and-plugin).

## <a id="tanzu-cli-command-groups"></a>Tanzu CLI command groups

Tanzu CLI commands are organized into command groups. View a list of available command groups by
running:

```console
tanzu
```

The list of command groups that you see depends on which CLI plug-ins are installed on your local
machine.

## <a id="install-new"></a> Install new plug-ins

Install the CLI plug-ins required for Tanzu Application Platform by running:

```console
tanzu plugin install --group vmware-tap/default:v{{ vars.tanzu-cli.plugin_group_version }}
```

Plug-ins for the Tanzu CLI are distributed by using a centralized plug-in repository.

The centralized plug-in repository contains:

- CLI plug-ins, which you can install individually
- Plug-in groups, which are a collection of plug-ins that you can install by using a single command

To discover and install plug-ins and plug-in groups:

1. Discover which plug-ins or plug-in groups are available for installation by running:

   ```console
   tanzu plugin search
   ```

   ```console
   tanzu plugin group search
   ```

   > **Note** Include the `--show-details` flag to see all plug-in or group versions available
   > for installation.

2. Install the latest version of a plug-in or plug-in group by running:

   ```console
   tanzu plugin install PLUG-IN-NAME
   ```

   ```console
   tanzu plugin install --group GROUP-NAME
   ```

3. Install a specific version of a plug-in or plug-in group by running:

   ```console
   tanzu plugin install PLUG-IN-NAME --version VERSION
   ```

   ```console
   tanzu plugin install --group GROUP-NAME:VERSION
   ```
