# Add stages to your Supply Chain

This topic tells you how to add a stage to a `SupplyChain` resource.

{{> 'partials/supply-chain/beta-banner' }}

## <a id="prep"></a> Preparation

You must have:

- A `SupplyChain` resource. For how to build one, see
  [Build your first SupplyChain](my-first-supply-chain.hbs.md).
- A component. For how to build one, see [Build your first component](my-first-component.hbs.md).

## <a id="add-stage"></a> Add a stage to an existing `SupplyChain`

This section describes how to add a stage to an existing `SupplyChain` resource if the `SupplyChain`
resource has not been deployed to clusters used by developers.

> **Important** If the `SupplyChain` resource is in use by developers with existing running
> workloads, do not add a stage. Instead, create a new `SupplyChain` resource with an updated version
> in the name, such as `MavenAppBuildv2`.
>
> Creating a new `SupplyChain` resource with every breaking Workload API change provides developers
> with a migration path and a chance to test their app on the new `SupplyChain` resource.

To add a stage:

1. Add a component to the `SupplyChain` resource by adding the following YAML snippet between
   `source-git-provider` and the `buildpack-build` stage in your `SupplyChain` resource:

    ```yaml
         - componentRef:
             name: COMPONENT-NAME
           name: unit-testing
    ```

    Where `COMPONENT-NAME` is the component name. For example, `maven-unit-tester-1.0.0`.

1. Update the `SupplyChain` resource by running:

   ```console
   NAMESPACE=mysupplychains make install
   ```

## <a id="useful-links"></a> Useful links

- [Component API Reference](../../reference/api/component.hbs.md)
- [Component Catalog](../../reference/catalog/about.hbs.md)