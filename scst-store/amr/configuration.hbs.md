# Configure Artifact Metadata Repository

This topic tells you how to configure Artifact Metadata Repository (AMR).

## <a id='amr-observer'></a> AMR Observer

You can obtain the Tanzu Application Platform values schema by running:

```console
tanzu package available get amr-observer.apps.tanzu.vmware.com/${VERSION} --values-schema \
--namespace tap-install
```

The following example is AMR Observer configuration that is located under the `amr` key in the
Tanzu Application Platform values file:

```yaml
amr:
  observer:
    location: |
      labels:
      - key: env
        value: prod
    resync_period: "10h"
    ca_cert_data: |
      -----BEGIN CERTIFICATE-----
      Custom CA certificate for AMR CloudEvent Handler's HTTPProxy with custom TLS certs
      -----END CERTIFICATE-----
    cloudevent_handler:
      endpoint: "https://amr-cloudevent-handler.DOMAIN"
      liveness_period_seconds: 10
    auth:
      kubernetes_service_accounts:
        enable: true
        autoconfigure: true
        secret:
          ref: "amr-observer-edit-token"
          value: ""
    max_concurrent_reconciles:
      image_vulnerability_scans: 1
```

Where `DOMAIN` is the domain you want to target.

Configuration options:

- `amr.observer.location`
  - Default value: `""`
  - Location is the multiline string configuration for the location content.
  - The YAML string can contain a single field:
    - `labels`, which consists of an array for a key and value pairing. It is useful for adding
      searchable and identifiable metadata. For enabling DORA functions, include a label named
      `env`. For more information, see
      [DORA metrics in Tanzu Developer Portal](../../tap-gui/plugins/dora.hbs.md).

- `amr.observer.resync_period`
  - Default value: `"10h"`
  - `resync_period` decides the minimum frequency at which watched resources reconcile. A lower
    period corrects entropy more quickly, but reduces responsiveness to change if there are many
    watched resources. Change this value with caution. It is 10 hours by default if unset.

- `amr.observer.ca_cert_data` or `shared.ca_cert_data`
  - Default value: `""`
  - `ca_cert_data` adds certificates to the `truststore` that `amr-observer` uses.

    ```console
    kubectl -n metadata-store get secrets/amr-cloudevent-handler-ingress-cert -o \
    jsonpath='{.data."crt.ca"}' | base64 -d
    ```

- `amr.observer.cloudevent_handler.endpoint`
  - Default value: `http://amr-cloudevent-handler.metadata-store.svc.cluster.local:80`
  - The URL of the AMR CloudEvent Handler endpoint.
  - On the Tanzu Application Platform View or Full profile cluster, obtain the AMR CloudEvent
    Handler ingress address to configure this property by running:

    ```console
    kubectl -n metadata-store get httpproxies.projectcontour.io amr-cloudevent-handler-ingress -o \
    jsonpath='{.spec.virtualhost.fqdn}'
    ```

  > **Note** Ensure that you set the correct protocol. If there is TLS, you must prepend `https://`.
  > If there is no TLS, you must prepend `http://`.

- `amr.observer.cloudevent_handler.liveness_period_seconds`
  - Default: `10`
  - This is the period in seconds between executed health checks to the AMR CloudEvent Handler endpoint.

- `amr.observer.auth.kubernetes_service_accounts`
  - `.enable`
    - Default value: `true`
    - This includes an authorization header when communicating with AMR CloudEvent Handler.
  - `.autoconfigure`
    - Default value: `true`
    - This delegates creation of an authentication token secret to the AMR. It is only applicable on
      Full and View clusters.
  - `.secret`
    - This is the secret with the access token for communicating with `cloudevent-handler`
    - `.ref`
      - Default value: `""`
      - This is the secret name that contains the access token.
    - `.value`
      - Default value: `""`
      - This is the secret as a plaintext string. This allows integration with TMC secret imports.

- `amr.observer.deployed_through_tmc`
  - Default value: `null`
  - The Tanzu Application Platform multicluster deployment happens through Tanzu Mission Control
    when you set `deployed_through_tmc` to `true`.
  - When deploying with TMC, `MultiClusterPropertyCollector` overwrites existing Observer package
    configuration values. For the workaround, see the
    [known issue](../../release-notes.hbs.md#1-10-0-scst-store-ki).

- `amr.observer.max_concurrent_reconciles`
  - This configures maximum concurrent reconciles for controllers.
  - `.image_vulnerability_scans`
    - Default value: `1`
    - This is the maximum concurrent reconciles for observing `ImageVulnerabilityScans`.

- `amr.observer.log_level`
  - Default value: `INFO`
  - This sets the log level. Set it to `DEBUG` to acquire more information through the logs.

## <a id='amr-graphql'></a> AMR GraphQL

- `amr.graphql.auth.kubernetes_service_accounts`
  - `.enable`
    - Default value: `true`
    - Enable authentication for the AMR GraphQL server. By default it is set to `true`.
  - `.autoconfigure`
    - Default value: `true`
    - This delegates creation of the authentication token secret to the AMR. By default it is set to
      `true`.

## <a id='amr-cloudevent-handler'></a> AMR CloudEvent Handler

- `amr.cloudevent_handler.auth.kubernetes_service_accounts`
  - `.enable`
    - Default value: `true`
    - This enables authentication and authorization for services accessing AMR.
  - `.autoconfigure`
    - Default value: `true`
    - This delegates creation of an authentication token secret to the AMR. By default it is set to
      `true`.

- `amr.cloudevent_handler.log_level`
  - Default value: `INFO`
  - This sets the log level. Set it to `DEBUG` to acquire more information through the logs.
