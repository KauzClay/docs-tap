# Add Stages to your Supply Chain

This topic tells you how to build a Component using Tanzu Supply Chain.

{{> 'partials/supply-chain/beta-banner' }}

## Prerequisite

- Ensure that you created a SupplyChain by following the tutorial [Build your first SupplyChain](./my-first-supply-chain.hbs.md).
- Ensure that you created a Component by following the tutorial [Build your first Component](./my-first-component.hbs.md).

## Add a stage to an existing Supplychain

>**Important** This is recommended only if this SupplyChain has not been deployed to clusters used by developers. If it is in use by developers with existing running workloads, this option is discouraged. Instead, you should create a new SupplyChain with an updated version in the name, for example, `MavenAppBuildv2`. Creating a new version of the SupplyChain with every breaking Workload API change provides developers with a migration path and a chance to test their app on the new version of the SupplyChain.

Let's add our `maven-unit-tester` Component to the `MavenAppBuildv1` SupplyChain.

We can do that by adding the following yaml snippet between the `source-git-provider` and `buildpack-build` stage in our `MavenAppBuildv1` SupplyChain:

```
     - componentRef:
         name: maven-unit-tester-1.0.0
       name: unit-testing
```

and then run the following command to update the Supplychain:
```
NAMESPACE=mysupplychains make install
```

## Useful links

- [Component API Reference](../../reference/api/component.hbs.md)
- [Component Catalog](../../reference/catalog/about.hbs.md)