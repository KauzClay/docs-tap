# Duck type resources

This topic tells you about the duck type resources for Tanzu Supply Chain.

The two duck type resources in Tanzu Supply Chain are [Workloads](workload.hbs.md) and
[WorkloadRuns](workloadrun.hbs.md).

In brief, a duck type resource is a resource with significant commonality. A duck type resource is
similar to a Class in object-oriented design, but without the inheritance.

![Diagram of a Workload Run. The workload properties are labeled as dynamic. The stages and conditions properties are labeled as static.](images/duck-type.png)

Some sections of a duck type API are unchanging (static) and others can and do change (dynamic).

Both Tanzu Supply Chain duck types have dynamic kinds, which means there can be many Kubernetes CRDs
that comply with the `WorkloadRun` duck type.