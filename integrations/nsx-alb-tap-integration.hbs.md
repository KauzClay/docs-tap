# (Beta) Configure Tanzu Application Platform and VMware NSX Advanced Load Balancer to support L7 routing to web workloads

This topic tells you how to configure Tanzu Application Platform (commonly known as TAP) with
VMware NSX Advanced Load Balancer (formerly known as Avi Networks) to support `web` workloads.

For information about VMware NSX Advanced Load Balancer (NSX ALB), see the
[VMware NSX Advanced Load Balancer documentation](https://docs.vmware.com/en/VMware-NSX-Advanced-Load-Balancer/index.html).

## <a id="overview"></a> Overview

This integration enables you to use NSX ALB to handle L7 traffic to Tanzu Application Platform `web`
workloads on a Tanzu Application Platform Run cluster.

Unlike in normal Tanzu Application Platform behavior or the
[NSX ALB L4 setup](../cloud-native-runtimes/how-to-guides/avi-cnr-integration.hbs.md),
the Contour Envoy is no longer part of the data path for cluster-external traffic. Instead,
cluster-external traffic travels from NSX ALB Service Engines directly to `web` workload pods on
the cluster.

Contour and Envoy are still used for handling cluster-local traffic to `web` workloads.

The integration with NSX ALB is only for `web` workloads. Support for Server Workloads is not
available in Tanzu Application Platform at this time.

Certain features of Cloud Native Runtimes are not supported in this integration, including
scale-to-zero and Knative `DomainMappings`.

This integration is only available on TKGm clusters currently.

> **Caution** Start from a new environment. This integration involves changing your ingress
> provider, and Tanzu Application Platform does not support no-downtime upgrades or switches to this
> configuration. The current Avi Controller might drop traffic when unexpected bursts occur.

## <a id="prepare"></a> Prepare

To prepare, obtain:

- An AVI Enterprise license
- AVI Controller v22.1 or later
- TKGm v2.5.1 or later
- GatewayAPI V1 CRDs (included with TKGm v2.5.1 and later)
- AKO v1.12 or later (included with TKGm)
- A domain that `web` workloads will be available under. This topic uses `DOMAIN` to represent this
  value.

### <a id="config-tkgm"></a> Configure TKGm to use NSX ALB as the load-balancer for all clusters

Before installing Tanzu Application Platform, the TKGm environment must be configured to use NSX ALB.

#### <a id="set-up-clstr-mngmnt"></a> Set up cluster management

Follow the steps in the
[Tanzu Kubernetes Grid documentation](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/2.5/tkg-deploy-mc/mgmt-reqs-network-nsx-alb-service.html#basic-setup-configure-nsx-alb-as-load-balancer-implementation-for-all-clusters-0)
to configure a TKGm management cluster to use NSX ALB.

#### <a id="crt-for-wrkld-clstr"></a> Create `ServiceEngineGroup` and `AKODeploymentConfig`

For each workload cluster, you must create:

- A `ServiceEngineGroup` in the Avi Controller
- An `AKODeploymentConfig` on the management cluster

For convenience, follow a consistent naming pattern. This topic uses:

- `wlN` as the cluster name, where `N` is a number
- `wlN-SEG` as the `ServiceEngineGroup` name
- `wlN-adc` as the `AKODeploymentConfig` file name

This topic mentions only one workload cluster, but the steps can be repeated as necessary for other
clusters.

1. Create a `ServiceEngineGroup` named `wl0-SEG`. For instructions, see the
   [Avi Load Balancer documentation](https://docs.vmware.com/en/VMware-Avi-Load-Balancer/30.2/Configuration-Guide/GUID-127D2E27-A75B-40E5-8F58-E12B6E57DC5D.html).

1. On the management cluster, create an `AKODeploymentConfig` file named `wl0-adc`. You can use the
   `install-ako-for-all` `AKODeploymentConfig` as a baseline, which is found on the management
   cluster by default. Ensure your new `AKODeploymentConfig` file has these fields set as follows:

   - `clusterSelector.matchLabels` = `cluster-type: wl0-gateway-clusterip`
   - `extraConfigs.featureGates.GatewayAPI` = `true`
   - `extraConfigs.serviceType` = `ClusterIP`

1. On the management cluster, create a workload cluster configuration YAML file with the following
   content:

    ```yaml
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

## <a id="config-for-wrkld-clstr"></a> Configure the workload cluster

The following procedures take place on the workload cluster. If you have multiple workload clusters,
repeat the procedures on each cluster.

### <a id="crt-rsrcs-wrkld-clstr"></a> Create the AVI Gateway API resources on the workload cluster

Now that AKO is configured to reconcile `GatewayAPI` resources, you need to create a `Gateway` and
`GatewayClass` for AVI on your workload cluster.

1. Targeting the workload cluster.
1. See if `GatewayClass` is on the cluster by running:

   ```console
   kubectl get gatewayclass
   ```

   If there is a `GatewayClass` on the cluster already, you don't need to create one. Otherwise,
   create the following:

    ```yaml
    ---
    apiVersion: gateway.networking.k8s.io/v1
    kind: GatewayClass
    metadata:
      name: avi-lb
    spec:
      controllerName: "ako.vmware.com/avi-lb"
    ---
    apiVersion: gateway.networking.k8s.io/v1
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

> **Caution** If you create this Gateway resource as part of a kapp app or package, you must include
> rebase rules so that the listeners can be updated freely. Otherwise kapp-controller might
> reconcile the Gateway resource and delete any changes that Tanzu Application Platform has
> programmed into the listeners.

### <a id="instll-cnfg-tap-clstr"></a> Install and configure Tanzu Application Platform on the workload cluster

While still targeting the workload cluster, it is time to install Tanzu Application Platform.

1. Create an overlay for the `cnrs` package by applying this YAML to the cluster:

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
         target-burst-capacity: "0"
    ```

1. Create a `tap-values.yaml` file with the following content. You can configure other Tanzu
   Application Platform components as you want here as well. This example shows only the content
   necessary for this integration:

    ```yaml
    shared:
      ingress_domain: DOMAIN

    profile: run
    ...
    cnrs:
      default_ingress_provider: gateway-api
      domain_template: "\{{.Name}}-\{{.Namespace}}.\{{.Domain}}"
      ingress_issuer: ""
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
          gateway:
            gatewayRef:
              name: tanzu-contour-gateway
              namespace: tanzu-system-ingress

    package_overlays:
    - name: cnrs
      secrets:
      - name: cnr-disable-scale-to-zero
    ```

   The objects referenced in `cnrs.gateway_api.internal` do not exist yet. You will create them later.

1. Using the values file you just created, follow the steps in
   [Install Tanzu Application Platform (online)](../install-online/intro.hbs.md).

#### <a id="cnfg-dmn-cnr-nsx-alb"></a> Configure the domain with CNR and NSX ALB

By default, CNR creates FQDNs for Knative Services that are in the pattern
`\{{.Name}}.\{{.Namespace}}.\{{.Domain}}`. However, in this example CNRs configuration, you set
`cnrs.domain_template = "\{{.Name}}-\{{.Namespace}}.\{{.Domain}}`.

You do this because Avi Kubernetes Operator (AKO) currently supports only wildcard domains in the
Gateway listener to a depth of 1.

For example, given a Gateway with a listener for `*.foo.com`, AKO matches `app.foo.com`, but not
`app.ns.foo.com`.

If you want to use the default Knative FQDN pattern, create a wildcard listener for each namespace
to which you want to deploy `web` workloads. For example, the following example accepts `web`
workloads in `ns1` and `ns2`, but not `ns3`:

```yaml
---
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: tanzu-avi-gateway
  namespace: avi-system
spec:
  gatewayClassName: avi-lb
  listeners:
   - name: http
     protocol: HTTP
     hostname: '*.ns1.DOMAIN'
     port: 80
     allowedRoutes:
       namespaces:
         from: All
   - name: http
     protocol: HTTP
     hostname: '*.ns2.DOMAIN'
     port: 80
     allowedRoutes:
       namespaces:
         from: All
```

> **Caution** When using wildcard listeners, do not use nested wildcard patterns. For example, if
> you add a listener with the hostnames `*.DOMAIN` and `*.ns1.DOMAIN`, the AVI controller will not
> accept routes with the domain `*.ns1.DOMAIN`.

### <a id="crt-gtwy-rsrcs-for-cntr"></a> Create the Gateway resources for Contour

Now that Tanzu Application Platform is installed on your Run Cluster, and Contour is running on the
cluster, create the Gateway resources for Contour as follows:

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

After doing this, you might need to restart the Contour pods in the `tanzu-system-ingress` namespace
for this Gateway to be detected.

> **Note** Though you need to create a `GatewayClass` resource for the Gateway to reference, Contour
> does not reconcile the `GatewayClass` resource when supporting `GatewayAPI` in Static mode. The
> status remains as `Unknown`, and that is okay.

## <a id="verify"></a> Verify Knative Serving for Cloud Native Runtimes

If you [verify that CNR is configured correctly with a simple Knative Service](../cloud-native-runtimes/how-to-guides/app-operators/verifying-serving.hbs.md),
your workload cluster is now configured to use NSX ALB as the ingress provider for `web` workloads.

## <a id="advncd-config"></a> Advanced Configuration

The following procedures are for advanced configurations of this integration.

### <a id="use-dflt-tls-scrt"></a> Use a default TLS secret

To make your `web` workloads available over HTTPS by using a wildcard certificate, modify your Avi
Gateway to supply a default TLS Certificate as a secret.

1. Create a secret in the `avi-system` namespace that holds your certificate. For instructions, see
   the [Kubernetes documentation](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_create/kubectl_create_secret_tls/).
   This example assumes the secret is named `defaultTLSSecret`:

    ```yaml
    ---
    apiVersion: gateway.networking.k8s.io/v1
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

1. Update `tap-values.yaml` with this additional CNR configuration:

    ```yaml
    cnrs:
      ...
      default_external_scheme: "https"
    ```

### <a id="config-gslb"></a> Configure global server load balancing (GSLB)

`web` workloads can be used as NSX ALB GSLB services under the following conditions:

- Each Tanzu Application Platform Run cluster that will be part of the GSLB is configured
  identically. Crucially, each cluster uses the same `DOMAIN`.
- The GSLB domain is the same as the `DOMAIN` on each of the clusters.
- `web` workloads that will be part of the same GSLB service are deployed in the same namespace
  across clusters.
- The GSLB FQDN matches the FQDN of the `web` workloads. For example, for the web workload `foo`
  deployed in the namespace `bar` on clusters A and B, with the domain `baz`:
  - Each web workload has the FQDN `foo.bar.baz`
  - The GSLB service has the FQDN `foo.bar.baz`

1. To enable GSLB in your AVI environment, follow the steps in the
   [NSX Advanced Load Balancer documentation](https://docs.vmware.com/en/VMware-NSX-Advanced-Load-Balancer/22.1/GSLB_Guide/GUID-C1E8D6E2-6753-4955-9511-847A36724B0F.html)
   and, for each site, ensure that the domain matches the `DOMAIN` used to configure your
   Tanzu Application Platform installation.

1. After your GSLB sites are configured, create a GSLB service by following the steps in the
   [NSX Advanced Load Balancer documentation](https://docs.vmware.com/en/VMware-NSX-Advanced-Load-Balancer/22.1/GSLB_Guide/GUID-67110CB8-2548-4F71-A234-4F24B2F02AC9.html)
   and, for each site, add a pool and then select the corresponding `VirtualService` for the Avi
   Gateway on the workload cluster.
