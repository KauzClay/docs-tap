# Configure workloads to use Enterprise Config Server

This topic tells you how to configure Tanzu Application Platform (commonly known as TAP) workloads
running Spring Boot applications to connect to Config Server resources.

## <a id='prepare'></a> Prepare

Create a `ConfigServer` resource. For instructions, see
[Create ConfigServer resources](create-configserver-resources.hbs.md).

## <a id="claim-creds"></a> Claim credentials

Claim credentials from Tanzu CLI or by creating a `ResourceClaim` directly:

Using Tanzu CLI
: Claim credentials by running:

   ```console
   tanzu service resource-claim create CLAIM-NAME \
       --resource-kind ConfigServer \
       --resource-api-version config-server.spring.tanzu.vmware.com/v1alpha1 \
       --resource-name RESOURCE-NAME \
       --resource-namespace RESOURCE-NAMESPACE \
       --namespace WORKLOAD-NAMESPACE
   ```

   Where:

   - `CLAIM-NAME` is a name you choose for your claim.
   - `RESOURCE-NAME` is the name of the `ConfigServer` resource you want to claim.
   - `RESOURCE-NAMESPACE` is the namespace that your `ConfigServer` resource is in.
   - `WORKLOAD-NAMESPACE` is the namespace that your workload is in.

   For example:

   ```console
   $ tanzu service resource-claim create cook \
       --resource-kind ConfigServer \
       --resource-api-version config-server.spring.tanzu.vmware.com/v1alpha1 \
       --resource-name cook-server \
       --resource-namespace cook \
       --namespace cook
   ```

Using a ResourceClaim
: Create a YAML file, similar to the following example, and name it `resource-claim.yaml`:

    ```yaml
    ---
    apiVersion: services.apps.tanzu.vmware.com/v1alpha1
    kind: ResourceClaim
    metadata:
      name: cook
      namespace: cook
    spec:
      ref:
        apiVersion: config-server.spring.tanzu.vmware.com/v1alpha1
        kind: ConfigServer
        name: cook-server
        namespace: cook
    ```

   In kubectl, create the `ResourceClaim` by running:

   ```console
   kubectl apply -f resource-claim.yaml
   ```

## <a id="inspect"></a> Inspect the progress of your claim

Inspect the progress of your claim creation by running a command in Tanzu CLI or kubectl:

- If using Tanzu CLI, run:

   ```console
   tanzu service resource-claim get MY-CLAIM-NAME --namespace MY-NAMESPACE
   ```

- If using kubectl, run:

   ```console
   kubectl get resourceclaim MY-CLAIM-NAME --namespace MY-NAMESPACE --output yaml
   ```

## <a id="inspect"></a> Use Config Server for workload configuration

To use Config Server for workload configuration:

1. If you have an existing application already configured to use
   [Spring Cloud Config Server](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/#_spring_cloud_config_client),
   claim `ConfigServer` credentials to access the running `ConfigServer` resources by adding the
   following to the `spec.serviceClaims` section of a workload:

    ```yaml
      serviceClaims:
        - name: cook
          ref:
            apiVersion: services.apps.tanzu.vmware.com/v1alpha1
            kind: ResourceClaim
            name: cook
    ```

   By claiming the credentials, a workload is configured to interact with the referenced Config
   Server.

   For example, you can use the following workload to deploy the
   [cook application](https://github.com/spring-cloud-services-samples/cook).

   `cook-workload.yaml` example:

    ```yaml
    # cook-workload.yaml
    ---
    apiVersion: carto.run/v1alpha1
    kind: Workload
    metadata:
      name: cook-workload
      namespace: cook
      annotations:
        kapp.k14s.io/change-rule.sa: "delete before deleting workloads-sa"
        kapp.k14s.io/change-group: "workload"
      labels:
        apps.tanzu.vmware.com/workload-type: web
        app.kubernetes.io/part-of: cook
    spec:
      build:
        env:
          - name: BP_JVM_VERSION
            value: "17"
          - name: BP_GRADLE_BUILD_ARGUMENTS
            value: "--no-daemon clean bootJar"
      params:
        - name: annotations
          value:
            autoscaling.knative.dev/minScale: "1"
      env:
        - name: SPRING_PROFILES_ACTIVE
          value: "development"
      serviceClaims:
        - name: cook
          ref:
            apiVersion: services.apps.tanzu.vmware.com/v1alpha1
            kind: ResourceClaim
            name: cook
      source:
        git:
          url: https://github.com/spring-cloud-services-samples/cook
          ref:
            #! TODO: We can switch back to 'main' branch when 'Tanzu Java Buildpack' for 1.10
            branch: kvmw/update
    ```

1. Create the workloads by running:

   ```console
   tanzu apps workload create -f WORKLOAD-NAME.yaml --yes
   ```

   Where `WORKLOAD-NAME` is the name of the workload, such as `cook-workload`.

1. From the Tanzu CLI, retrieve the ingress route associated with your application by running:

   ```console
   tanzu apps workload get WORKLOAD-NAME
   ```

   Where `WORKLOAD-NAME` is your workload name, such as `cook-workload`.

   For example:

   ```console
   $ tanzu apps workload get cook-workload

   # other output...

   🚢 Knative Services
      NAME            READY   URL
      cook-workload   Ready   https://cook-workload.cook.tap

   To see logs: "tanzu apps workload tail cook-workload --timestamp --since 1h"
   ```

   Where `https://cook-workload.cook.tap` is the accessible ingress route to the cook application

1. Visit `[ROUTE]/restaurant`, where `[ROUTE]` is the ingress route you just retrieved. The
   `special` of the day is taken from the configuration repository and the value of `cook.special`.

## <a id="exec-jar-file-app"></a> (Optional) Use Config Server with an executable JAR file application

In the `cook` application example, `BP_GRADLE_BUILD_ARGUMENTS` is set to include the `bootJar`
task in addition to the default Gradle build arguments. This setting is necessary for this example
base because the `build.gradle` file contains a `jar` section, and the Spring Boot buildpack will
not inject the `spring-cloud-bindings` library into the application if it is an executable JAR file.

`spring-cloud-bindings` is required to process the `serviceClaim` into properties that tell the
discovery client how to find the Config Server.

To use Config Server with an executable JAR file application, explicitly include
[spring-cloud-bindings v2.0.3](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-bindings/2.0.3)
or later and set the `org.springframework.cloud.bindings.boot.enable=true` system property as
described in the [library README file](https://github.com/spring-cloud/spring-cloud-bindings#spring-boot-configuration)
in GitHub.
