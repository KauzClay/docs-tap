# Overview of Components

This topic tells you about the `Component` resource in Tanzu Supply Chain.
For reference information, see [Component API](../../reference/api/component.hbs.md).

{{> 'partials/supply-chain/beta-banner' }}


![Diagram of the relationships between key Tanzu Supply Chain resources. Some resources are grouped together as namespace-scoped. Other resources are grouped together as cluster-scoped.](./images/core-concepts-component.jpg)

Components encapsulate the work to be performed in composable and reusable pieces.
Components are analogous to steps, stages, jobs, and tasks in other CI/CD offerings.
Components are different to other CI/CD offerings in three distinct ways:

- Component configuration requirements are declared, static, and enforced. The configuration is used
  to build a `Workload` resource that is strongly-typed and well-documented.

- Components are observed by users through a strict error-reporting API.

- Components can exhibit reentrant behavior. If you define a resumption for a component, the
  resumption can trigger new workload runs. This keeps the stored information about the component in
  transient dependencies within the component. For more information, see [Resumptions](resumptions.hbs.md).

These design constraints exist to:

- Simplify the end-user experience by providing a single, well-defined API and a way of mitigating
  errors if they arise.

- Simplify the authoring experience by requiring only minimal Kubernetes experience to construct
  supply chains that meet your organization's needs.

<!--
[SupplyChain]: ./supply-chains.hbs.md
[SupplyChains]: ./supply-chains.hbs.md
[Workload]: ./workloads.hbs.md
[Workloads]: ./workloads.hbs.md
[WorkloadRuns]: ./workload-runs.hbs.md
[WorkloadRun]: ./workload-runs.hbs.md
[Resumptions]: ./resumptions.hbs.md
[Resumption]: ./resumptions.hbs.md
-->
