# Recommendations for upgrading Kubernetes

This topic gives you recommendations for what to do before upgrading a Kubernetes cluster.

## <a id='upgrade-k8s'></a> Before upgrading Kubernetes

Starting from Tanzu Application Platform v1.11.0, all Knative components have pod anti-affinity rules
set in their deployments.
This ensures that pods are evenly distributed across different nodes, reducing the risk of outages.

This configuration applies to all Knative services. It ensures that pods from the same
revision are preferably scheduled on different nodes. These measures improve the
Kubernetes upgrade experience by minimizing the chances of traffic disruptions during upgrades.

Before starting your Kubernetes upgrade process, VMware recommends that you:

- Ensure that your Knative services have a minimum of two pods per active revision.
  You can achieve this by annotating the Knative service.
  For more information, see the [Kubernetes documentation](https://knative.dev/docs/serving/autoscaling/scale-bounds/#lower-bound).
- Provision a new set of Kubernetes nodes.
- Cordon and drain your Kubernetes nodes one at a time.
