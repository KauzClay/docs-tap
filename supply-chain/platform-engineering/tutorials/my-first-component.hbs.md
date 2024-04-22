# Build your first Component

This topic tells you how to build a Component using Tanzu Supply Chain.

{{> 'partials/supply-chain/beta-banner' }}

## Prerequisite

- Ensure that you created a SupplyChain by following the tutorial [Build your first SupplyChain](./my-first-supply-chain.hbs.md).

## Getting started

In the first tutorial, we built a SupplyChain that retrieves the source code from a Git repository, and proceeds with building and
packaging it as a Carvel package. Subsequently, it initiates a pull request to transfer the Carvel
package to a GitOps repository, facilitating the installation of the built package on the Run clusters. 


In this tutorial, We will create a component that we can use in our supplychain for unit testing our Maven apps. 
We will create a component called `maven-unit-tester` and that will run unit tests on the source pulled by the `source-git-provider` stage.

```
tanzu supplychain component get source-git-provider-1.0.0 --show-details
```
Example: 
```
...
📤 Outputs
   git
    ├─ digest: $(resumptions.check-source.results.sha)
    ├─ type: git
    └─ url: $(resumptions.check-source.results.url)
   source
    ├─ digest: $(pipeline.results.digest)
    ├─ type: source
    └─ url: $(pipeline.results.url)
...
```



## Useful links

- [Component API Reference](../../reference/api/component.hbs.md)
- [Component Catalog](../../reference/catalog/about.hbs.md)