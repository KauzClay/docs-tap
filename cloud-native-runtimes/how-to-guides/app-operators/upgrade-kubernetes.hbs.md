# Upgrading Kubernetes Recommendations

This topic informs you about recommendations prior to upgrading a Kubernetes cluster.

## <a id='upgrade-k8s'></a> Before upgrading Kubernetes

Starting from TAP 1.11.0, all Knative components have pod anti-affinity rules set in their deployments.
This ensures that pods are evenly distributed across different nodes, reducing the risk of outages.

Additionally, this configuration applies to all Knative services, ensuring that pods from the same revision are preferably
scheduled on different nodes. These measures are designed to improve the Kubernetes upgrade experience by minimizing the chances of traffic disruptions during upgrades.

Before starting your Kubernetes upgrade process, we recommend taking the following measures:

- Ensure that your Knative services have a minimum of 2 pods per active revision. You can achieve this by annotating the Knative service. See the [documentation](https://knative.dev/docs/serving/autoscaling/scale-bounds/#lower-bound) for details.
- Provision a new set of Kubernetes nodes.
- Cordon and drain your Kubernetes nodes one at a time.
