# Configure CORS policy for Cloud Native Runtimes

This topic tells you how to establish a cross-origin resource sharing (CORS) policy for
Contour HTTPProxy to enable cross-domain requests from trusted sources.
The CORS policy applies to all external routes for web workloads on the cluster.

To configure a CORS policy in Cloud Native Runtimes:

1. Add `cnrs.cors_policy` to your `tap-values.yaml` configuration file. For example:

    ```yaml
    cnrs:
      cors_policy: |
        allowOrigin:
          - "*" # allows any origin
        allowMethods:
          - GET
          - POST
          - OPTIONS
    ```

    For information about the available configuration options, see the [Contour documentation](https://projectcontour.io/docs/main/config/cors/).

1. Update your Tanzu Application Platform installation by running:

    ```console
    tanzu package installed update tap \
      --package tap.tanzu.vmware.com \
      --version ${TAP_VERSION} \
      --values-file tap-values.yaml \
      --namespace tap-install
    ```
