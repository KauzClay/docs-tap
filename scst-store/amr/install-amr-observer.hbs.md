# Artifact Metadata Repository Observer for Supply Chain Security Tools - Store

This topic tells you how to install Artifact Metadata Repository (AMR) Observer for Supply Chain
Security Tools (SCST) - Store.

## <a id='prerecs'></a> Before you begin

The AMR Observer is deployed by default on the Tanzu Application Platform Full, Build, and Run
profiles.

## <a id='switch-context'></a> Switch context

If AMR Observer is installed on a separate cluster from AMR CloudEvent Handler, target the correct
cluster before updating package values by running:

1. Switch context by running:

   ```console
   kubectl config use-context OBSERVER-CLUSTER-NAME
   ```

   Where `OBSERVER-CLUSTER-NAME` is the name of the cluster you want to use.

1. Update `tap-values.yaml` for your desired configuration.

1. Update the installed Tanzu Application Platform package on the cluster by running:

   ```console
   tanzu package installed update tap --values-file tap-values.yaml -n tap-install
   ```

## <a id='install'></a> Configure AMR Observer in a multicluster deployment

When you install AMR Observer on a different cluster from the AMR CloudEvent Handler, the following
values are required:

- `amr.observer.cloudevent_handler.endpoint`, which is required for the Observer to send to the AMR
  CloudEvent Handler.
- `amr.observer.cloudevent_handler.ca_cert_data` or `shared.ca_cert_data`, which is required for AMR
  CloudEvent Handlers that use Custom CA certificates to generate the associated TLS certificate for
  the ingress endpoint.

> **Note** If SCST - Scan 2.0 is installed after AMR Observer has already been deployed, you must
> restart AMR Observer to observe the new `ImageVulnerabilityScan` Custom Resource that was
> installed with SCST - Scan 2.0.

1. Configure AMR Observer by running:

   ```console
   kubectl -n amr-observer-system rollout restart deployment amr-observer-controller-manager
   ```

1. Verify that the following log appears if the AMR Observer is observing the
   `ImageVulnerabilityScan` Custom Resource:

   ```log
   2023-06-28T17:56:43Z  INFO  Starting Controller  {"controller": "imagevulnerabilityscan", "controllerGroup": "app-scanning.apps.tanzu.vmware.com", "controllerKind": "ImageVulnerabilityScan"}
   ```

For information about logging, see
[Troubleshoot - AMR Observer Logs](troubleshooting.hbs.md#amr-observer-logs) and
[Configuration - AMR Observer](configuration.hbs.md#amr-observer).

## <a id='standalone-install'></a> Install standalone AMR Observer

To install AMR Observer outside a Tanzu Application Platform profile:

1. See the available version by running:

   ```console
   tanzu package available list amr-observer.apps.tanzu.vmware.com -n tap-install
   ```

   For example:

   ```console
   $ tanzu package available list amr-observer.apps.tanzu.vmware.com -n tap-install

     NAME                                VERSION        RELEASED-AT
     amr-observer.apps.tanzu.vmware.com  0.7.0          2024-06-13 21:49:57 -0400 EDT
   ```

1. Look at the package `values-schema` needed to create the values file by running:

   ```console
   tanzu package available get amr-observer.apps.tanzu.vmware.com/VERSION --values-schema --namespace tap-install
   ```

   Where `VERSION` is the version you saw earlier.

   For example:

   ```console
   $ tanzu package available get amr-observer.apps.tanzu.vmware.com/0.7.0 --values-schema --namespace tap-install
   KEY                                                           DEFAULT                                                            TYPE     DESCRIPTION
   deployed_through_tmc                                          false                                                              boolean  TAP Multi Cluster deployment will happen through Tanzu Mission Control when
                                                                                                                                             `deployed_through_tmc` is set to true
   observer.app_limit_cpu                                        500m                                                               string   CPU limit for AMR Observer manager container
   observer.location                                             ""                                                                 string   location is the multiline string configuration for the location.conf content.
   observer.resync_period                                        10h                                                                string   resync_period determines the minimum frequency at which watched resources
                                                                                                                                             are reconciled. A lower period will correct entropy more quickly, but reduce
                                                                                                                                             responsiveness to change if there are many watched resources. Change this value
                                                                                                                                             only if you know what you are doing. Defaults to 10 hours if unset.
   observer.ca_cert_data                                         ""                                                                 string   ca_cert_data is used to add certificates to the truststore that is used by the
                                                                                                                                             amr-observer.
   observer.cloudevent_handler.endpoint                          http://amr-cloudevent-handler.metadata-store.svc.cluster.local:80  string   The URL of the cloudevent handler endpoint.
   observer.cloudevent_handler.liveness_period_seconds           10                                                                 integer  The period in seconds between executed health checks to the cloudevent handler
                                                                                                                                             endpoint.
   observer.log_level                                            INFO                                                               string   Log level (INFO or DEBUG)
   observer.max_concurrent_reconciles.image_vulnerability_scans  1                                                                  integer  Max concurrent reconciles for observing ImageVulnerabilityScans.
   observer.app_limit_memory                                     1Gi                                                                string   Memory limit for AMR Observer manager container
   observer.app_req_cpu                                          100m                                                               string   CPU request for AMR Observer manager container
   observer.app_req_memory                                       512Mi                                                              string   Memory request for AMR Observer manager container
   observer.auth.kubernetes_service_accounts.secret.value        ""                                                                 string   Secret as a plain text string. This allows integrating with Tanzu Mission
                                                                                                                                             Control secret imports.
   observer.auth.kubernetes_service_accounts.secret.ref          amr-observer-edit-token                                            string   Secret name which contains the access token.
   observer.auth.kubernetes_service_accounts.autoconfigure       true                                                               boolean  Delegate creation of auth token secret to AMR Observer. By default it is set to
                                                                                                                                             true.
   observer.auth.kubernetes_service_accounts.enable              true                                                               boolean  Include Authorization header when communicating with AMR CloudEvent Handler.
   ```

   For more information, see
   [Configure Artifact Metadata Repository](configuration.hbs.md#Configuration).

1. Install the package by running:

   ```console
   tanzu package install
   ```
