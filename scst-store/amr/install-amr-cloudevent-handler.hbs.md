# Install Artifact Metadata Repository CloudEvent Handler

This topic tells you how to install the Artifact Metadata Repository (AMR) CloudEvent Handler.

## <a id='switch-context'></a>Switching Context

If AMR CloudEvent Handler is installed on a separate cluster, such as with a view profile cluster,
it is important that you target the correct cluster when updating. Ensure that the correct cluster
is targeted before updating package values.

1. Switch the context to the View profile cluster by running:

   ```console
   kubectl config use-context VIEW-CLUSTER-NAME
   ```

2. Update `tap-values.yaml` in an editor according to the desired configuration.

3. Update the installed Tanzu Application Platform package on the cluster by running:

   ```console
   tanzu package installed update tap --values-file tap-values.yaml -n tap-install
   ```

   Where `VIEW-CLUSTER-NAME` is the name of the view profile cluster you want to use.

## <a id='install'></a> Install

The AMR CloudEvent Handler is installed by default in the Tanzu Application Platform Full and View
profiles.
