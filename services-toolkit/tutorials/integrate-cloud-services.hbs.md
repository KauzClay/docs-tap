# Integrate cloud services into Tanzu Application Platform

In this Services Toolkit tutorial you learn how [service operators](../reference/terminology-and-user-roles.hbs.md#so)
can integrate the cloud services of their choice into Tanzu Application Platform (commonly known as TAP).

There are a multitude of cloud-based services available on the market for consumers today.
AWS, Azure, and GCP all provide support for a wide range of fully-managed, performant, and
on-demand services. These services range from databases, to message queues, to storage solutions and
beyond.
In this tutorial you learn how to integrate any one of these services into Tanzu Application Platform,
so that you can offer it for application teams to consume in a simple and effective way.

This tutorial is written at a slightly higher level than the other tutorials in this documentation.
This is because it is not feasible to write detailed, step-by-step documentation for integrating
every cloud-based service into Tanzu Application Platform.
Each service brings a different set of considerations and concerns.

Instead, this tutorial guides you through the general approach to integrating cloud-based services into
Tanzu Application Platform.
While specific configuration changes between services, the steps in the process remain the same.
The aim is to give you enough understanding so that you can integrate any cloud-based service
you want into Tanzu Application Platform.

For a more specific and low-level procedure, see
[Configure dynamic provisioning of AWS RDS service instances](../how-to-guides/dynamic-provisioning-rds.hbs.md),
which provides each step in detail for integrating AWS RDS.
It might be useful to read through that guide even if you want to integrate with one of the other
cloud providers.

In Tanzu Application Platform v1.7 or later, you can instead use the [AWS Services](../../aws-services/about.hbs.md)
package, which provides a more streamlined approach for integrating services from AWS into
Tanzu Application Platform.

## <a id="about"></a> About this tutorial

**Target user role**:       Service operator<br />
**Complexity**:             Advanced<br />
**Estimated time**:         30 minutes<br />
**Topics covered**:         Dynamic provisioning, cloud-based services, AWS, Azure, GCP, Crossplane<br />
**Learning outcomes**:      An understanding of the steps involved to integrate cloud-based services
into Tanzu Application Platform<br />

## <a id="concepts"></a> Concepts

The following is a high-level workflow outlining what is required to integrate a cloud-based service
into Tanzu Application Platform.

1. [Install a Provider and create a ProviderConfig](#install-provider):

    - Follow the official Upbound documentation to install the `Provider` and create a `ProviderConfig`.

1. [Create a CompositeResourceDefinition](#create-xrd):

    - Create a `CompositeResourceDefinition` to define the shape of a new API type representing the service.
    - Choose which configuration parameters to expose to apps teams, if any.

1. [Create a Composition](#create-composition):

    - Create a `Composition` using managed resources supplied by the `Provider`.
    - You can compose as many or as few managed resources as required to generate a service instance
      that application workloads can connect to and use over the network.
    - (Optional but recommended) Configure the connection secret to adhere to the service binding
      specification for Kubernetes.

1. [Create a provisioner-based ClusterInstanceClass](#clusterinstanceclass):

    - Create a provisioner-based `ClusterInstanceClass` pointing to the `CompositeResourceDefinition`
      created earlier.

1. [Create the required RBAC](#configure-rbac):

    - Create RBAC to allow claiming from the class by using the `claim` verb pointing to the
      provisioner-based `ClusterInstanceClass`.

1. [Verify your integration by creating a ClassClaim](#verify):

    - Create a `ClassClaim` pointing to the provisioner-based `ClusterInstanceClass` to begin a dynamic
      provisioning request.
    - Wait for the `ClassClaim` to report `READY=True`.

## <a id="procedure"></a> Procedure

This tutorial provides the steps required to integrate cloud services and includes tips and references
to example configurations where appropriate.

### <a id="install-provider"></a> Step 1: Install a `Provider`

Install a Crossplane `Provider` for your cloud of choice. Upbound provides support for the
three main cloud providers:

- [provider-aws](https://marketplace.upbound.io/providers/upbound/provider-aws/latest)
- [provider-azure](https://marketplace.upbound.io/providers/upbound/provider-azure/latest)
- [provider-gcp](https://marketplace.upbound.io/providers/upbound/provider-gcp/latest)

> **Note** These cloud-based `Providers` often install hundreds of additional CRDs onto the cluster.
> This can have a negative impact on cluster performance.
> For more information, see [Cluster performance degradation due to a large number of CRDs](../../crossplane/reference/known-limitations.hbs.md#too-many-crds).

Choose the `Provider` you want, and then follow Upbound's official documentation to install the
`Provider` and to create a corresponding `ProviderConfig`.

> **Important** The official documentation for the `Provider` includes a step to install Universal Crossplane.
> You can skip this step because Crossplane is already installed as part of Tanzu Application Platform.
>
> The `Provider` documentation also assumes Crossplane is installed in the `upbound-system` namespace.
> However, when working with Crossplane on Tanzu Application Platform, it is installed to the
> `crossplane-system` namespace by default.
> Ensure that you use the correct namespace when you create the `Secret` and the `ProviderConfig`
> with credentials for the `Provider`.

### <a id="create-xrd"></a> Step 2: Create a `CompositeResourceDefinition`

Create a `CompositeResourceDefinition`. This defines the shape of a new API type to create the
cloud-based resources.

For help creating the `CompositeResourceDefinition`, see the [Crossplane documentation](https://docs.crossplane.io/latest/concepts/composition/#defining-composite-resources)
or see [Create a CompositeResourceDefinition](../how-to-guides/dynamic-provisioning-rds.hbs.md#compositeresourcedef)
in _Configure dynamic provisioning of AWS RDS service instances_.

### <a id="create-composition"></a> Step 3: Create a `Composition`

This step is likely to be the most time-consuming. The `Composition` is where you define the configuration
for the resources that make the service instances for app teams to claim.
This configures the resources required for service instances that users can connect to and use over
the network.

To get started with creating a `Composition`, first read about configuring the composition in the
[Upbound documentation](https://docs.crossplane.io/v1.11/concepts/composition/#configuring-composition).

For examples, you can also see [Example Compositions](../reference/example-compositions.hbs.md).

### <a id="clusterinstanceclass"></a> Step 4: Create a provisioner-based `ClusterInstanceClass`

Create a provisioner-based `ClusterInstanceClass` that is configured to refer to the
`CompositeResourceDefinition` created earlier. For example:

```yaml
---
apiVersion: services.apps.tanzu.vmware.com/v1alpha1
kind: ClusterInstanceClass
metadata:
  name: cloud-service-foo
spec:
  description:
    short: FooDB by cloud provider Foo!
  provisioner:
    crossplane:
      compositeResourceDefinition: XRD-NAME
```

Where `XRD-NAME` is the name of your `CompositeResourceDefinition`.

For an example, see [Make the service discoverable](../how-to-guides/dynamic-provisioning-rds.hbs.md#make-discoverable)
in _Configure dynamic provisioning of AWS RDS service instances_.

### <a id="configure-rbac"></a> Step 5: Configure RBAC

Create a Role-Based Access Control (RBAC) rule that uses the `claim` verb and points to the
`ClusterInstanceClass` you created. For example:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: app-operator-claim-foo-db
  labels:
    apps.tanzu.vmware.com/aggregate-to-app-operator-cluster-access: "true"
rules:
- apiGroups:
  - "services.apps.tanzu.vmware.com"
  resources:
  - clusterinstanceclasses
  resourceNames:
  - cloud-service-foo
  verbs:
  - claim
```

For an example, see [Configure RBAC](../how-to-guides/dynamic-provisioning-rds.hbs.md#configure-rbac)
in _Configure dynamic provisioning of AWS RDS service instances_.

### <a id="verify"></a> Step 6: Verify your integration

To test your integration, create a `ClassClaim` that points to the `ClusterInstanceClass` you created.
For example:

```yaml
---
apiVersion: services.apps.tanzu.vmware.com/v1alpha1
kind: ClassClaim
metadata:
  name: claim-1
spec:
  classRef:
    name: cloud-service-foo
  parameters:
    key: value
```

Verify that the `ClassClaim` eventually transitions into a `READY=True` state.
If it doesn't, debug the `ClassClaim` using kubectl.
For how to do this, see [Debug ClassClaim and provisioner-based ClusterInstanceClass](../how-to-guides/troubleshooting.hbs.md#debug-dynamic-provisioning).
