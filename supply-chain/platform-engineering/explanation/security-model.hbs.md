# Security Model

This topic tells you about the security model for Tanzu Supply Chain. This security model is for
executing stages of a Supply Chain where workloads exist in a separate namespace to the Supply
Chain.

{{> 'partials/supply-chain/beta-banner' }}


![Diagram of the Security Model that depicts the cluster scope, the supply chain namespace, and the workload namespace.](./images/security-model.png)

Runs for associated workloads are created in the same namespace as the workload.

`Stages`, `Resumptions`, `TaskRuns`, and `PipelineRuns` are created by default in the Supply Chain
namespace. This gives the components in these stages visibility over platform secrets in the Supply
Chain namespace.

If you want a stage of a pipeline to execute in the workload namespace, use the
`securityContext.runAs` setting. For example, you can allow a source component to retrieve the
source from a developer-controlled repository by using secrets that the developer provides in the
workload namespace.

For more information, see the `securityContext`
[specification](../../reference/api/supplychain.hbs.md#security-context) in the `SupplyChain` API
topic.
