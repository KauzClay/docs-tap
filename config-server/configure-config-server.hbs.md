# Create ConfigServer resources

This topic tells you about options when creating a `ConfigServer` resource.

## <a id="discover-params"></a> Discover available parameters

Examine the available parameters when creating a `ConfigServer` resource by running:

```console
kubectl explain configserver.spec
```

For example:

```console
$ kubectl explain configserver.spec
KIND:     ConfigServer
VERSION:  config-server.spring.tanzu.vmware.com/v1alpha1

RESOURCE: spec <Object>

DESCRIPTION:
     ConfigServerSpec defines the desired state of ConfigServer

FIELDS:
   backends	<[]Object> -required-
     List of backends used by the config server. There must be at least one
     backend for config-server.

   replicas	<integer> -required-
     Number of desired config server replicas. Defaults to 1.

   resources	<Object>
     The config server compute resource requirements

   tls	<Object>
     TLS configuration for the config server
```

Here is a second example describing the git configuration fields:

```console
kubectl explain configserver.spec.backends.git
```

For example:

```console
$ kubectl explain configserver.spec.backends.git 
KIND:     ConfigServer
VERSION:  config-server.spring.tanzu.vmware.com/v1alpha1

RESOURCE: git <Object>

DESCRIPTION:
     Git backend configuration

FIELDS:
   basicAuth	<Object>
     For HTTP/S addresses, the credentials used to access the Git repository if
     protected by HTTP Basic authentication. Optional.

   defaultLabel	<string>
     The default label used if a request is received without a label. Optional:
     Defaults to main.

   paths	<[]string>
     A list of patterns used to search for configuration-containing
     subdirectories in the Git repository. Optional.

   proxy	<Object>
     The Proxy configuration for the Git repository. Optional.

   skipTLSVerify	<boolean>
     For HTTPS addresses, whether to skip validation of the SSL certificate on
     the Git repository's server. Optional: Defaults to false.

   ssh	<Object>
     For SSH addresses, the credentials used to access the Git repository if
     protected by SSH. Optional.

   timeout	<integer>
     Number of seconds that the config server will wait to acquire a connection
     to the Git repository. Optional: Defaults to 5 seconds.

   ttl	<integer>
     Number of seconds to wait before updating the repository clone from Git
     repository, when a client requests configuration. Optional: Defaults to 0
     seconds. Default value (0) means the repository clone is updated every time
     a client requests configuration. Negative value means the repository clone
     will not be updated, after it is cloned.

   uri	<string> -required-
     The HTTP/S or SSH address of the Git repository.

```

## <a id="create-configserver"></a> Create a ConfigServer resource

To create a `ConfigServer` resource for using the Spring Cloud Services [cook](https://github.com/spring-cloud-services-samples/cook)
 demo application, in a namespace called cook:

1. Create a `ConfigServer` resource by using the following YAML definition:

    ```yaml
    ---
    apiVersion: config-server.spring.tanzu.vmware.com/v1alpha1
    kind: ConfigServer
    metadata:
      name: cook-server
      namespace: cook
    spec:
      replicas: 1
      tls:
        activated: true
      backends:
        - git:
          uri: https://github.com/spring-cloud-services-samples/cook-config
    ```

1. Save the YAML definition as `configserver.yaml`.

1. Apply the YAML definition by running:

   ```console
   kubectl apply -f configserver.yaml
   ```

1. Check the status of the `ConfigServer` resource by running:

   ```console
   kubectl describe configserver RESOURCE-NAME
   ```

   For example:

   ```console
   $ kubectl describe configserver cook-server --namespace cook
    Name:         cook-server
    Namespace:    cook
    API Version:  config-server.spring.tanzu.vmware.com/v1alpha1
    Kind:         ConfigServer
    Metadata:
      Creation Timestamp:  2024-04-30T18:17:59Z
      Generation:          1
      Resource Version:  3871989
      UID:               ecd86b7a-d9fa-4999-8607-b023d39cb255
    Spec:
      Backends:
	Git:
	  Uri:   https://github.com/spring-cloud-services-samples/cook-config
      Replicas:  1
      Resources:
	Limits:
	  Cpu:     2
	  Memory:  1Gi
	Requests:
	  Cpu:     256m
	  Memory:  512Mi
      Tls:
	Activated:  true
    Status:
      Binding:
	Name:  cook-server-service-binding-v5xlr
      Conditions:
	Last Transition Time:  2024-04-30T18:18:04Z
	Message:               Configuration has been resolved successfully.
	Reason:                ConfigResolutionSuccess
	Status:                True
	Type:                  ConfigResolution
	Last Transition Time:  2024-04-30T18:18:55Z
	Message:               Deployment has minimum availability.
	Reason:                MinimumReplicasAvailable
	Status:                True
	Type:                  DeploymentReady
	Last Transition Time:  2024-04-30T18:18:55Z
	Message:               
	Reason:                Ready
	Status:                True
	Type:                  Ready
      Configuration Secret:
	Name:  cook-server-configuration-4bt9v
      Deployment:
	Config Hash:     14247363468230849770
	Ready Replicas:  1/1
      Image Pull Secret:
	Name:               cook-server-registry-credentials-vbgx7
      Observed Generation:  1
      Service:
	Name:  cook-server-f6l9v
      Tls:
	Issuer Name:  cook-server-9d67p
   ```

   A successful `ConfigServer` resource has a `Ready` condition set to `true` and a
   `status.binding.name` field pointing to a secret containing connection information 
   which can be used by service bindings. 
