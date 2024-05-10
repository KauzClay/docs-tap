# Configure TAP and VMware NSX Advanced Load Balancer to Support L7 Routing to Web Workloads

This topic tells you how to configure the Tanzu Application Platform (TAP) with VMware NSX Advanced Load Balancer (NSX ALB, formerly known as Avi Networks) to support Web Workloads.

For information about VMware NSX Advanced Load Balancer, see the [VMware NSX Advanced Load Balancer documentation](https://docs.vmware.com/en/VMware-NSX-Advanced-Load-Balancer/index.html).

## Overview
This integration allows you to use NSX ALB to handle L7 traffic to TAP Web Workloads on a TAP Run Cluster. Unlike normal TAP behavior, or in the [NSX ALB L4 setup](../cloud-native-runtimes/how-to-guides/avi-cnr-integration.hbs.md), Contour’s Envoy is no longer part of the data path for cluster-external traffic. Instead, cluster-external traffic will travel from NSX ALB Service Engines directly to Web Workload pods on the cluster.

>**Note**: Contour and Envoy are still used for handling cluster-local traffic to Web Workloads.

>**Note**: This integration is only for Web Workloads. Support for Server Workloads is not available in TAP at this time.

>**Note**: Certain features of Cloud Native Runtimes are not supported in this integration, including scale-to-zero and Knative DomainMappings.

>**Note**: This integration is only available on TKGm clusters currently.

>**Warning**: This integration involves changing your ingress provider. We do not support no-downtime upgrades/switches to this configuration. It is recommended to start from a new/fresh environment.

## Prerequisites

* AVI Enterprise License
* AVI Controller 22.1+
  * TODO: fill in with Beta release of AVI
* TKGm 2.5.1+
* GatewayAPI V1 CRDs (comes with TKGm 2.5.1+)
* AKO 1.12+ (comes with TKGm)
* A domain which Web Workloads will be available under. This document will use `DOMAIN` to represent this value.

### Configuring TKGm to use NSX ALB as Load Balancer Implementation for All Clusters

Before installing TAP, the TKGm environment must be configured to use NSX ALB.


#### Management Cluster Setup

Follow the [Basic Setup step in these TKGm docs](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/2.5/tkg-deploy-mc/mgmt-reqs-network-nsx-alb-service.html#basic-setup-configure-nsx-alb-as-load-balancer-implementation-for-all-clusters-0) to get a TKGm management cluster configured to use NSX ALB.

#### Workload Cluster Creation

For each workload cluster, you must create:
1. A ServiceEngineGroup in the Avi Controller.
2. An AKODeploymentConfig (ADC) on the management cluster.


For convenience, consider following a consistent naming pattern. This doc use:
* Cluster Name: `wlN` where `N` is a number
* ServiceEngineGroup name: `wlN-SEG`
* ADC file name: `wlN-adc`

This guide will only use 1 workload cluster, though the steps can be repeated as necessary.

1. Create the ServiceEngineGroup named `wl0-SEG` (see [here](https://docs.vmware.com/en/VMware-Avi-Load-Balancer/30.2/Configuration-Guide/GUID-127D2E27-A75B-40E5-8F58-E12B6E57DC5D.html) for instructions).
  
2. On the management cluster, create the ADC named `wl0-adc`. You can use the `install-ako-for-all` ADC (found on the management cluster by default) as a baseline. Make sure your new ADC has the following field set:
  a. `clusterSelector.matchLabels` = `cluster-type: wl0-gateway-clusterip`
  b. `extraConfigs.featureGates.GatewayAPI` = `true`
  c. `extraConfigs.serviceType` = `ClusterIP`
  d. For more advanced configurations, such as `dataNetwork`, consult the [TKG and NSX ALB documentation](TODO where to link to).

3. On the management cluster, create a workload cluster configuraiton file with the following contents:
   
   ```
   # These are required for this guide
   CLUSTER_NAME: wl0
   AVI_LABELS: '{"cluster-type": "wl0-gateway-clusterip"}'

   # These values may be updated depending on your preferences
   CLUSTER_PLAN: dev
   CNI: antrea
   INFRASTRUCTURE_PROVIDER: vsphere
   KUBERNETES_VERSION: v1.28.7+vmware.1
   OS_ARCH: amd64
   OS_NAME: ubuntu
   OS_VERSION: '22.04'
   ```

After creating a new workload cluster, you should be able to get a Kubeconfig and access it.

## Configuring the Workload Cluster

The rest of this guide will be done on the workload cluster. If you have multiple workload clusters, these steps need to be done on each of them.

### Creating the AVI Gateway API Resources on the Workload Cluster

Now that AKO is configured to reconcile GatewayAPI resources, you need to create a `Gateway` and `GatewayClass` for AVI on your workload cluster.


Make sure you are targeting the workload cluster, and create the following:

```yaml
---
apiVersion: gateway.networking.k8s.io/v1beta1
kind: GatewayClass
metadata:
  name: avi-lb #! this may be created automatically when AKO is configured
spec:
  controllerName: "ako.vmware.com/avi-lb"
---
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: tanzu-avi-gateway
  namespace: avi-system
spec:
  gatewayClassName: avi-lb
  listeners:
   - name: http
     protocol: HTTP
     hostname: '*.DOMAIN'
     port: 80
     allowedRoutes:
       namespaces:
         from: All
```

Where `DOMAIN` is the `DOMAIN` listed in the prerequisites.

>**Note**: If you create this Gateway resource as part of a Kapp App or Package, you must include rebase rules so that the listeners can be updated freely. We do not want Kapp-controller re-reconciling and wiping away any changes that TAP has programmed into the listeners.

### Installing and Configuring TAP on the Workload Cluster
Still targeting the workload cluster, it is time to install TAP.

First, we need to create an overlay for the `cnrs` package:

```yaml
apiVersion: v1
kind: Secret
metadata:
 name: cnr-disable-scale-to-zero
 namespace: tap-install
stringData:
 cnr-disable-scale-to-zero.yml: |
   #@overlay/match by=overlay.subset({"metadata":{"name":"config-autoscaler","namespace":"knative-serving"}})
   ---
   data:
     #@overlay/match missing_ok=True
     enable-scale-to-zero: "false"
     #@overlay/match missing_ok=True
     min-scale: "1"
     #@overlay/match missing_ok=True
     target-burst-capacity: "0" #! needed until Avi supports backend header manipulation
```

Apply this to the cluster.

Next, create a `tap-values.yaml` file with the following content. You may configure other TAP components as you wish here as well, this example only shows the necessary bits for this integration:

```yaml
shared:
  ingress_domain: DOMAIN
  ingress_issuer: "" 

profile: run
...
cnrs:
  default_ingress_provider: gateway-api
  gateway_api:
    external:
      class: avi-lb
      gateway: avi-system/avi-gateway
    internal:
      class: tanzu-contour
      gateway: tanzu-system-ingress/tanzu-contour-gateway
      service: tanzu-system-ingress/envoy
contour:
  contour:
    configFileContents:
      #! enableExternalNameService: true
      gateway:
        gatewayRef:
          name: tanzu-contour-gateway
          namespace: tanzu-system-ingress

package_overlays:
- name: cnrs
  secrets:
  - name: cnr-disable-scale-to-zero

```

>**Note**: The objects referenced in `cnrs.gateway_api.internal` do not exist yet. You will create them in the next step.

Using the values file you just created, proceed to [install TAP Run](../install-online/intro.hbs.md).

### Creating the Gateway Resources for Contour

Now that TAP is installed on your Run Cluster, and Contour is running on the cluster, create the Gateway resources for Contour:

```yaml
---
kind: GatewayClass
apiVersion: gateway.networking.k8s.io/v1
metadata:
  name: tanzu-contour
spec:
  controllerName: projectcontour.io/gateway-controller
---
kind: Gateway
apiVersion: gateway.networking.k8s.io/v1
metadata:
  name: tanzu-contour-gateway
  namespace: tanzu-system-ingress
spec:
  gatewayClassName: tanzu-contour
  listeners:
    - name: http
      protocol: HTTP
      port: 80
      allowedRoutes:
        namespaces:
          from: All
```

After doing this, you may need to roll the Contour pods in the tanzu-system-ingress namespace for this Gateway to get picked up.

>**Note**: though we need to create a GatewayClass for the Gateway to reference, Contour doesn’t actually reconcile the GatewayClass resource when supporting GatewayAPI in Static mode (source). The status will remain in an Unknown state, that is okay.


## Validation

At this point, your workload cluster should be configured to use NSX ALB as the ingress provider for Web Workloads.

You should now be able [verify CNRS is configured correctly with a simple Knative Service](../cloud-native-runtimes/how-to-guides/app-operators/verifying-serving.hbs.md).

Alternatively, [create sample Web Workload](TODO do we have a link for this?).

## Advanced Configurations

The remainder of this document will go into more advanced configurations of this integrations.

### Using a Default TLS Secret

If you wish to make your Web Workloads available over HTTPS using a wildcard certificate, you can modify your Avi Gateway to supply a default TLS Certificate as a Secret, like this

```yaml
---
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: tanzu-avi-gateway
  namespace: avi-system
spec:
  gatewayClassName: avi-lb
  listeners:
   - name: https
     protocol: HTTP
     port: 443
     hostname: '*.DOMAIN'
     tls:
       certificateRefs:
       - name: defaultTLSSecret
         namespace: avi-system
         kind: Secret
     allowedRoutes:
       namespaces:
         from: All
```

Where `DOMAIN` is `DOMAIN` listed in the prerequisites.

Additionally, you will need to update your `tap-values.yaml` with this additional CNR configuration:

```yaml
cnrs:
  ...
  default_external_scheme: "https"
```

### GSLB Configuration

Web Workloads can be used as NSX ALB GSLB Services under the following requirements:
* Each TAP Run cluster that will be part of the GSLB should be configured identically
   * Crucially, each cluster must use the same DOMAIN
* The GSLB domain must be the same as the DOMAIN on each of the clusters
* Web Workloads intended to be part of the same GSLB service must be deployed in the same namespace across clusters.
* The GSLB FQDN must match the FQDN of the Web Workloads. Example:
   * Given Web Workload foo deployed in namespace bar on clusters A and B, with a Domain of `baz`:
      * Each Web Workload will have an FQDN of `foo.bar.baz`
      * The GSLB Service must have the FQDN `foo.bar.baz`


Under these conditions, you can enable GSLB in your AVI environment.
[The NSX ALB docs include instructions for setting up a GSLB Site](https://docs.vmware.com/en/VMware-NSX-Advanced-Load-Balancer/22.1/GSLB_Guide/GUID-C1E8D6E2-6753-4955-9511-847A36724B0F.html), which involves:
* Creating a DNS virtual service
* Enabling GSLB on leader sites
* Adding follower sites
* Makes sure the domain matches the DOMAIN used to configure your TAP install

Assuming your GSLB sites are configured, you can [create a GSLB Service](https://docs.vmware.com/en/VMware-NSX-Advanced-Load-Balancer/22.1/GSLB_Guide/GUID-67110CB8-2548-4F71-A234-4F24B2F02AC9.html).

For each site, add a pool and select the corresponding VirtualService for the Avi Gateway on the workload cluster.
