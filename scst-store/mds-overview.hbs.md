# Overview of the Metadata Store

This topic gives you an overview of the Metadata Store.

> **Important**  The Metadata Store component is deprecated. VMware plans to remove it in a future
> Tanzu Application Platform version.

## <a id='overview'></a> Overview

The Metadata Store saves software bills of materials (SBoMs) to a database and
allows you to query for image, source code, package, and vulnerability
relationships. The Metadata Store integrates with
[Supply Chain Security Tools - Scan](../scst-scan/overview.hbs.md) to automatically
store the resulting source code and image vulnerability reports. The Metadata Store
accepts CycloneDX input and outputs in both human-readable and machine-readable formats,
including JSON, text, and CycloneDX.

## <a id='multicluster-config'></a> Multicluster configuration

You can configure the Metadata Store in a multicluster setup.
For more information, see [Multicluster setup for Supply Chain Security Tools - Store](multicluster-setup.hbs.md).

## <a id='integrate'></a> Integrating with Tanzu Developer Portal

By using Supply Chain Choreographer in Tanzu Developer Portal, you can
visualize your supply chain. Supply Chain Choreographer uses the Metadata Store
to show the packages and vulnerabilities in your source code and images.

## <a id='additional-info'></a>Additional documentation

For more information about the API, deployment details and configuration, AWS Relational Database Service (RDS) configuration, other database backup recommendations, and known issues, see [Additional documentation](additional.hbs.md).
