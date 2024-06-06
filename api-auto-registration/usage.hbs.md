# Use API Auto Registration

This topic describes how you can use API Auto Registration.

## <a id='overview'></a> Overview

The run profile requires you to [update the install values](./configuration.hbs.md#update-values)
before proceeding. For iterate and full profiles, the default values work but you
might prefer to update them. For information about profiles,
see [About Tanzu Application Platform profiles](../about-package-profiles.hbs.md#profiles-and-packages).

## <a id='prerecs'></a> Prerequisites

API Auto Registration requires the following:

1. A location exposing a dynamic or static API specification.

2. An APIDescriptor Custom Resource (CR) with that location created in the cluster.

3. (Optional) Configurations:

   1. Cross-Origin Resource Sharing (CORS) for OpenAPI specifications
   2. Connect the component to the auto-registered API in Tanzu Developer Portal

## <a id='config'></a> Configurations

To generate OpenAPI Spec:

- [By creating a simple Spring Boot app](#using-simple-app)

- [By scaffolding a new project using App Accelerator Template](#using-app-acc-template)

- [In an existing Spring Boot project](#existing-spring-project)

To create APIDescriptor Custom Resource:

- [Using Out Of The Box Supply Chains](#using-ootb-supply-chain)

- [Using Custom Supply Chains](#using-custom-supply-chain)

- [Using other GitOps processes or Manually](#using-gitops-manually)

Additional configurations:

- [CORS for viewing OpenAPI Spec in Tanzu Developer Portal](#cors)

- [Connection between the Component and API in Tanzu Developer Portal](#component)

## <a id='generate-openapi'></a> Generate OpenAPI specifications

This section tells you how to generate OpenAPI specifications.

### <a id='using-simple-app'></a>Using a Spring Boot app with a REST service

You can use a [Spring Boot example app](https://github.com/making/rest-service) built using
[Building a RESTful Web Service guide](https://spring.io/guides/gs/rest-service/).
and has the [Springdoc dependency](https://springdoc.org/#getting-started).

Example of a workload using the Spring Boot app:

```yaml
apiVersion: carto.run/v1alpha1
kind: Workload
metadata:
  name: simple-rest-app
  labels:
    ...
    apis.apps.tanzu.vmware.com/register-api: "true"
spec:
  source:
    ...
  params:
    - name: api_descriptor
      value:
        type: openapi
        location:
          path: "/v3/api-docs"
        system: dev
        owner: team-a
        description: "A set of API endpoints."
```

### <a id='using-app-acc-template'></a>Using App Accelerator template

If you are creating a new application exposing an API, you might use the [java-rest-service](https://github.com/vmware-tanzu/application-accelerator-samples/tree/main/java-rest-service)
App Accelerator template to get a pre-built app that includes a `workload.yaml` with a basic REST
API. From your Tanzu Developer Portal Accelerators
tab, search for the accelerator and scaffold it according to your needs.

### <a id='existing-spring-project'></a>Using an existing Spring Boot project using springdoc

If you have an existing Spring Boot app that exposes an API, you can generate OpenAPI specifications
using springdoc. See the [springdoc documentation](https://springdoc.org/#getting-started)

After you have springdoc configured and an OpenAPI automatically generated, you can choose one of the
following methods to create the APIDescriptor custom resource:

- VMware recommends having your Spring Boot app to be managed using Workloads and the Out-Of-The-Box
(OOTB) supply chain. See the [Use Out-Of-The-Box (OOTB) supply chains](#using-ootb-supply-chain) for further instructions.
- To use custom supply chains, see [Using Custom Supply Chains](#using-custom-supply-chain).
-To use a different Gitops process or manage the APIDescriptor CR manually,
see the [Using other GitOps processes or Manually](#using-gitops-manually) section.


## <a id='create-api-descriptor'></a> Create APIDescriptor custom resource

This section tells you how to create an APIDescriptor custom resource.

### <a id='using-ootb-supply-chain'></a> Use Out-Of-The-Box (OOTB) supply chains

All the Out-Of-The-Box (OOTB) supply chains are modified so that they can use API Auto Registration.
If you want your workload to be auto registered, you must make modifications to your
workload YAML:

1. Add the label `apis.apps.tanzu.vmware.com/register-api: "true"`.
2. Add a parameter of `type api_descriptor`:

    ```yaml
      params:
        - name: api_descriptor
          value:
            type: openapi   # We currently support any of openapi, aysncapi, graphql, grpc
            location:
              path: "/v3/api-docs"  # The path to the api documentation
              baseURL: "http://my-spec.url" # Optional: The base URL to the api documentation if not served from the API runtime
            owner: team-petclinic   # The team that owns this
            system: petclinic       # The Backstage system entity this API belongs to
            description: "A set of API endpoints to manage the resources within the petclinic app."
    ```

The default supply chains use Knative to deploy your applications and are referenced as the
server location for the generated APIDescriptor.

For API specifications, there are 2 options for the location:

- If the API specifications are generated and served from your application endpoint,
the only location information you must set is the path to the API documentation.
The controller can figure out the base URL by using the Knative server reference.
- If your API specifications are served from a static location, you can hardcode the URL using the
`location.baseURL` property.

Example workload that exposes the API specifications from a Knative service:

```yaml
apiVersion: carto.run/v1alpha1
kind: Workload
metadata:
  name: petclinic-knative
  labels:
    ...
    apis.apps.tanzu.vmware.com/register-api: "true"
spec:
  source:
    ...
  params:
    - name: api_descriptor
      value:
        type: openapi
        location:
          path: "/v3/api-docs"
        system: pet-clinics
        owner: team-petclinic
        description: "A set of API endpoints to manage the resources within the petclinic app."
```

Example of a workload with a hardcoded URL to the API documentation:

```yaml
apiVersion: carto.run/v1alpha1
kind: Workload
metadata:
  name: petclinic-hard-coded
  labels:
    ...
    apis.apps.tanzu.vmware.com/register-api: "true"
spec:
  source:
    ...
  params:
    - name: api_descriptor
      value:
        type: openapi
        location:
          baseURL: http://petclinic-hard-coded.my-apps.tapdemo.vmware.com/
          path: "/v3/api-docs"
        owner: team-petclinic
        system: pet-clinics
        description: "A set of API endpoints to manage the resources within the petclinic app."
```

After the supply chain runs, it creates an `APIDescriptor` custom resource. Tanzu Application Platform uses this resource to auto register your API.
See [APIDescriptor explained](./key-concepts.hbs.md#api-descriptor).

### <a id='using-custom-supply-chain'></a>Using custom supply chains

If you are creating custom supply chains, you can still use API Auto Registration. To write a
supply chain pipeline, use `ClusterConfigTemplate` by the name of `config-template` in
your pipeline. To write a custom task, verify how the template is written to read parameters,
interpret baseURL from Knative Services, and construct APIDescriptor CRs.

In the Delivery pipeline, you must directly create an APIDescriptor custom resource. You must grant
permissions to create the CR from the delivery pipeline.

For information about APIDescriptors, see [APIDescriptor explained](./key-concepts.hbs.md#api-descriptor).

### <a id='using-gitops-manually'></a>Using other GitOps processes or manually

Using your GitOps process, or manually, you must stamp out an APIDescriptor CR and apply it in the
cluster you choose. Specify all the required fields for an APIDescriptor CR to reconcile.

For information about APIDescriptors, see [APIDescriptor explained](./key-concepts.hbs.md#api-descriptor).

## <a id='additional-config'></a> Additional configuration

This section tells you how to perform additional configuration.

### <a id='cors'></a>Setting up CORS for OpenAPI specifications

The agent, usually a browser, uses the [CORS](https://fetch.spec.whatwg.org/#http-cors-protocol)
protocol to verify whether the current origin uses an API.
To use the `Try it out`` feature for OpenAPI specifications from the API Documentation plug-in, you must
configure CORS to allow successful requests.

Your API must be configured to allow CORS Requests from Tanzu Developer Portal. How you
accomplish this varies based on your programming language and framework.
If you are using Spring, see [CORS support in spring framework](https://spring.io/blog/2015/06/08/cors-support-in-spring-framework).

At a high level, your API must accept the Tanzu Developer Portal domain must be accepted as a valid cross-origin.

To do this:

- **Origins allowed** header: `Access-Control-Allow-Origin`: A list of comma-separated values.
This list must include your Tanzu Developer Portal host.
- **Methods allowed** header: `Access-Control-Allow-Method`: Must allow the method used by your API.
Also confirm that your API supports preflight requests, a valid response to the OPTIONS HTTP method.
- **Headers allowed** header: `Access-Control-Allow-Headers`: If the API requires any header, you
must include it in the API configuration or your authorization server.

### <a id='component'></a>Connecting the component to the API in Tanzu Developer Portal

When the API-producing component is added to the Tanzu Developer Portal catalog, it is helpful
to connect the component to the API. By connecting the two entities, the catalog graph visualizes
this relationship for users. To connect the entities
you must add the `.spec.providesApis` in your component.yaml where you can list all the APIs it provides using string reference
format. Ensure that you follow the `namespace/name` format instead of `name`, otherwise the namespace
defaults to the component's namespace, typically `default`. The name is the auto-registered
API's name. VMware recommends updating the `component.yaml` with the API name after the auto-registration finishes.

The following is an example of a `component.yaml` with string entity reference to an API:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: petclinic
  namespace: petclinic-systems
  description: Petstore
spec:
  type: service
  lifecycle: dev
  owner: team-petclinic
  providesApis:
    - internal/petclinic-api-dev
```

For more information, see the [API documentation plug-in in Tanzu Developer Portal](../tap-gui/plugins/api-docs.hbs.md).