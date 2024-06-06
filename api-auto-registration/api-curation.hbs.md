# API Curation (alpha)

This topic tells you how to use API Curation to expose several standalone APIs as one API for
API Auto Registration.

> **Caution** The API Curation feature is in alpha and is intended for evaluation and test purposes
> only. VMware discourages using API Curation in a production environment.

## <a id='overview'></a> Overview

To unlock the maximum power of API curation, VMware recommends using a supported route provider.
Without this a supported route provider, the generated API specifications do not have a functional
server URL set for testing the curated API. You must manually create the routing resources to route
traffic to each referenced API to match with the aggregated API specifications. With the route
provider integration, these routing concerns are taken care of automatically.

## <a id='prerecs'></a> Prerequisites

A successful API curation requires the following prerequisites:

- (Optional) Spring Cloud Gateway for Kubernetes is installed.
- (Optional) `SpringCloudGateway` resources created with the matching `groupId` and `version`.
- One or more `APIDescriptor`s in the ready state.
- Create a `CuratedAPIDescriptor` resource referencing ready `APIDescriptor`s.

You can get the aggregated API specifications from the OpenAPI endpoint from the controller.

## <a id='create-route-provider'></a>(Optional) Install route provider and create gateway resources

You can optionally install a route provider and create gateway resources.

### <a id='setup-scg'></a>Setup Spring Cloud Gateway integration

Using Spring Cloud Gateway for Kubernetes (SCG) integration as a route provider is optional.
To install SCG, see [Install Spring Cloud Gateway for Kubernetes](../spring-cloud-gateway/install-spring-cloud-gateway.hbs.md).

For SCG v2.1.0 and later, if you enabled TLS on SCG, or installed it in a custom namespace,
you must overwrite `route_provider.spring_cloud_gateway.scg_openapi_service_url` in your
API Auto Registration values file.

```yaml
route_provider:
  spring_cloud_gateway:
    scg_openapi_service_url: "http://scg-openapi-service.spring-cloud-gateway.svc.cluster.local" # default value
```

If you are using an earlier version of SCG, consider upgrading or see the following table to
understand the impact:

| Capability with SCG v2.1.0 and later | Behavior before SCG v2.1.0 |
| --- | --- |
| The default value `scg_openapi_service_url` is sufficient if using the default SCG installation | You must overwrite the value `scg_openapi_service_url` with `http://scg-operator.tap-install.svc.cluster.local` if using the default SCG installation. |
| Matching SCG updates API metadata automatically and the generated OpenAPI specifications reflect the metadata | API metadata annotations are added or updated, but the API specifications exposed from SCG OpenAPI endpoint do not reflect that |

### <a id='create-scg'></a>Create SpringCloudGateway resource

You can create, using GitOps or manually, an `SpringCloudGateway` CR and apply it in the
cluster you choose.

The following is an example SCG resource:

```yaml
apiVersion: "tanzu.vmware.com/v1"
kind: SpringCloudGateway
metadata:
  name: test-api-curation
spec:
  api:
    version: 1.2.3
    groupId: test-api-curation
    serverUrl: https://my-curated-api.mydomain.com
    cors:
      allowedOrigins:
        - "http://api-portal.mydomain.com"
      allowedMethods:
        - "GET"
        - "PUT"
        - "POST"
      allowedHeaders:
        - '*'
```

The `groupId` and `version` matches on SCG when it reconciles for `CuratedAPIDescriptor`.
After finding a match, SCG uses `serverUrl` as the baseURL of the curated API.

## <a id='api-descriptors-curation'></a>Create APIDescriptors for curation

To create `APIDescriptor` resources, see [API Auto Registration Usage Guide](./usage.hbs.md). If any
of the referenced `APIDescriptor` are not ready, the `CuratedAPIDescriptor` keeps retrying until all
the referenced `APIDescriptor`s are ready. If the API specifications from any of the `APIDescriptor`
update, the controller selects the changes on the next reconciliation loop.

> **Note** By default, the update might take up to 10 minutes, including maximum 5 minutes to
> refresh the `APIDescriptor`, and maximum 5 minutes to refresh `CuratedAPIDescriptor`. You can
> shorten the wait time by configuring `sync_period` in the AAR values file.

## <a id='curated-api-descriptor'></a>Create CuratedAPIDescriptor custom resource

Using your GitOps process, or manually, you can create an CuratedAPIDescriptor CR and apply it in the
cluster you choose. Specify all the required fields for an CuratedAPIDescriptor CR to reconcile.

For information about CuratedAPIDescriptors, see
[CuratedAPIDescriptor explained](./key-concepts.hbs.md#curated-api-descriptor).

You can view the readiness of the applied CuratedAPIDescriptors by running:

```console
kubectl get curatedapidescriptors -A

NAMESPACE           NAME         GROUPID            VERSION   STATUS   CURATED API SPEC URL
my-apps             petstore     cute-api-group     1.2.3     Ready    http://AAR-CONTROLLER-FQDN/openapi/my-apps/petstore
```

## <a id='view-curated-api-desc'></a>View the auto-registered API within Tanzu Developer Portal

After you the CuratedAPIDescriptor you created is in the `Ready` state, navigate to Tanzu Developer
Portal to view the automatically registered APIs. You will see one new API for the
CuratedAPIDescriptor and the other APIs generated from APIDescriptors.

Additionally, you will see a component registered for the curated API. If you chose to use Spring
Cloud Gateway as the route provider in [Setup Spring Cloud Gateway integration](#setup-scg) earlier,
this component can show any gateway resources with the matching
`app.kubernetes.io/part-of={PART_OF}` label under the **Runtime Resources**. To make this
connection, add the `app.kubernetes.io/part-of={PART_OF}` label to the `SpringCloudGateway` instance
you create. For more information, see the
[Spring Cloud Gateway documentation](https://docs.vmware.com/en/VMware-Spring-Cloud-Gateway-for-Kubernetes/{{
vars.api-auto-registration.scg-version }}/scg-k8s/GUID-guides-tap-organization-catalog.html).

![Tanzu Developer Portal showing the running Spring Cloud Gateway resources.](./images/api-curation-component.png)

## <a id='retrieve-api-specs'></a>Retrieve curated API specifications

The API Auto Registration controller offers an endpoint to retrieve all the generated API
specifications for the curated APIs in the cluster.

To retrieve curated API specifications:

1. Find the HTTPProxy that's created to access the endpoint:

   ```console
   kubectl get httpproxy api-auto-registration-controller -n api-auto-registration
   ```

   The following is an example output:

   ```console
   NAME                               FQDN                              TLS SECRET                   STATUS   STATUS DESCRIPTION
   api-auto-registration-controller   api-auto-registration.tap.domain  api-auto-registration-cert   valid    Valid HTTPProxy
   ```

1. To get all the curated API specifications with curl, use FQDN with an http scheme:

   ```console
   curl http(s)://AAR-CONTROLLER-FQDN/openapi
   ```

  Where `AAR-CONTROLLER-FQDN` is the AAR FQDN controller you want the specifications for.

1. Retrieve specifications for `CuratedAPIDescriptor` from a specific namespace by specifying the
   following path:

   ```console
   curl http(s)://AAR-CONTROLLER-FQDN/openapi/NAMESPACE
   ```

   Where `NAMESPACE` is the namespace that you want to retrieve specifications from.

1. Retrieve specifications for a specific `CuratedAPIDescriptor` by `namespace` and `name`:

   ```console
   curl http(s)://AAR-CONTROLLER-FQDN/openapi/NAMESPACE/NAME
   ```

   Where:

   - `NAMESPACE` is the namespace of the `CuratedAPIDescriptor` that you want to use.
   - `NAME` is name of the `CuratedAPIDescriptor` that you want to use.

1. Retrieve specifications for a specific `groupId` and `version` combination by specifying query
   parameters:

   ```console
   curl http(s)://AAR-CONTROLLER-FQDN/openapi?groupId=GROUP-ID&version=VERSION
   curl http(s)://AAR-CONTROLLER-FQDN/openapi/NAMESPACE?groupId=GROUP-ID&version=VERSION
   ```

   Where:

   - `NAMESPACE` is the namespace that you want to retrieve specifications from.
   - `VERSION` is the version you want the specifications for.
   - `GROUP-ID` is group ID you want the specifications for.
   - `AAR-CONTROLLER-FQDN` is the AAR-CONTROLLER-FQDN you want to query.

1. Add the curated APIs to an API portal for display by configuring the source URL locations of an
   existing API portal. You can add all your curated APIs by using the unfiltered URL
   `http(s)://AAR-CONTROLLER-FQDN/openapi` or the filtered URL with query parameters to add a
   specific curated API of your choice. See
   [Configuring API portal for VMware Tanzu on Kubernetes](https://docs.vmware.com/en/API-portal-for-VMware-Tanzu/1.4/api-portal/GUID-configuring-k8s-basics.html#modifying-openapi-source-url-locations).

1. This curated API is not automatically registered in Tanzu Developer Portal. You can do this
   manually by creating an `api.yaml` and adding it to the catalog. For information about the
   structure of the definition file for an API entity, see the [API documentation plug-in in Tanzu Developer Portal](../tap-gui/plugins/api-docs.hbs.md)
   or the [Backstage Kind: API documentation](https://backstage.io/docs/features/software-catalog/descriptor-format#kind-api).
