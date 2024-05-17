# Configure CORS policy for Cloud Native Runtimes

Beginning with TAP 1.10, you can establish a CORS (Cross-origin resource sharing) policy for Contour HTTPProxy's to enable cross-domain requests from trusted sources.
Once set, the CORS policy will apply to all external routes for Web Workloads on the cluster.

To configure a CORS policy in Cloud Native Runtimes, you must adjust your configuration values file as demonstrated below.
For further details on available configurations, refer to the [Contour documentation](https://projectcontour.io/docs/main/config/cors/).

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

Update your Tanzu Application Platform installation:

   ```console
   tanzu package installed update tap -p tap.tanzu.vmware.com -v ${TAP_VERSION} --values-file tap-values.yaml -n tap-install
   ```
