# Supply Chain Security Tools - Scan 2.0 Observability

This topic guides you through observing Supply Chain Security Tools (SCST) - Scan 2.0. This helps
you understand each step of scanning.

## <a id="steps"></a> Scanning Steps

This section describes all the scanning steps and corresponding observability methods.

1. Watch the status of the scanning custom resources and child resources by running:

   ```console
   kubectl get -l imagevulnerabilityscan pipelinerun,taskrun,pod
   kubectl get imagevulnerabilityscan
   ```

1. View the status, reason, and URLs by running:

   ```console
   kubectl get imagevulnerabilityscan -o wide
   ```

1. View the complete status and events of scanning custom resources by running:

   ```console
   kubectl describe imagevulnerabilityscan IMAGE-VULNERABILITY-SCAN-NAME
   ```

   Where `IMAGE-VULNERABILITY-SCAN-NAME` is the name of an `ImageVulnerabilityScan` resource you
   want to inspect.

1. List the child resources of a scan by running:

   ```console
   kubectl get -l imagevulnerabilityscan=$NAME pipelinerun,taskrun,pod
   ```

1. Get the logs of the controller by running:

   ```console
   kubectl logs -f deployment/app-scanning-controller-manager -n app-scanning-system -c manager
   ```