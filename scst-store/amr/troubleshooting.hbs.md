# Troubleshooting Artifact Metadata Repository (AMR)

This topic tells you how to troubleshoot issues with Artifact Metadata Repository (AMR).

## <a id='debug'></a> Debug AMR

1. Pause reconciliation of the package by running:

   ```console
   tanzu package installed pause amr-observer -n tap-install
   # OR
   kctrl package installed pause -i amr-observer -n tap-install
   ```

1. Change the zap log level by running:

   ```console
   kubectl -n amr-observer-system \
   patch deployment amr-observer-controller-manager \
     --type='json' \
     -p='[{"op": "add",
           "path": "/spec/template/spec/containers/1/args/-",
           "value": "--zap-log-level=3"
         }]'
   ```

   Logs now show `LEVEL(-3)` with the example patch seen earlier. For example:

   ```console
   $ kubectl -n amr-observer-system logs deployments/amr-observer-controller-manager
   2023-06-20T15:42:39Z  LEVEL(-3)  httpclient.circuitbreaker  AMR CloudEventHandler  {"availability": true, "State": "closed"}
   ```

1. Unpause reconciliation of the package by running:

   ```console
   tanzu package installed kick amr-observer -n tap-install
   # OR
   kctrl package installed kick -i amr-observer -n tap-install
   ```

## <a id='resource-issue'></a> Resource Issue

If AMR Observer frequently restarts, it might have insufficient allocated memory resources. An
example symptom of this is an error log reporting failing leader election:

```console
E0530 05:01:20.297972       1 leaderelection.go:367] Failed to update lock: Put "https://10.0.0.1:443/apis/coordination.k8s.io/v1/namespaces/amr-observer-system/leases/3ed94067.gitlab.eng.vmware.com": dial tcp 10.0.0.1:443: i/o timeoutI0530 05:01:20.298127       1 leaderelection.go:283] failed to renew lease amr-observer-system/3ed94067.gitlab.eng.vmware.com: timed out waiting for the condition
{"level":"error","ts":"2024-05-30T05:01:20.298180924Z","logger":"main.setup","caller":"amr-observer/main.go:294","msg":"problem running manager","error":"leader election lost","stacktrace":"main.main\n\tamr-observer/main.go:294\nruntime.main\n\truntime/proc.go:271"} {"level":"info","ts":"2024-05-30T05:01:20.298227224Z","caller":"manager/internal.go:581","msg":"Stopping and waiting for non leader election runnables"}
```

To verify that the issue is insufficient allocated memory resources, increase the memory limit and
see if there are fewer restarts. To increase the memory limit, configure the following Tanzu
Application Platform values as shown:

```yaml
amr:
    observer:
        app_limit_memory: 2500Mi
```

For more configuration information, see
[Install standalone AMR Observer](install-amr-observer.hbs.md#standalone-install).

## <a id='health-check'></a> Health Check

AMR Observer does not send events to AMR CloudEvent Handler if the AMRCloudEvent Handler, AMR, or
Metadata Store is not working.

Sample log in AMR Observer when `HealthCheck` is working:

```log
2023-06-20T15:51:50Z  INFO  httpclient.circuitbreaker  Received response with status  {"status": "204 No Content"}
2023-06-20T15:51:50Z  INFO  httpclient.circuitbreaker  Successfully sent CloudEvent
```

Sample log in AMR Observer when `HealthCheck` is failing:

```log
2023-06-20T19:01:58Z  INFO  httpclient.circuitbreaker  Received response with status  {"status": "503 Service Unavailable"}
2023-06-20T19:01:58Z  ERROR  httpclient    {"error": "request failed with status 503"}
gitlab.eng.vmware.com/tanzu-image-signing/amr-observer/internal/cloud/event/client.(*CloudEventClient).logAndReturnError
  /workspace/internal/cloud/event/client/client.go:53
gitlab.eng.vmware.com/tanzu-image-signing/amr-observer/internal/cloud/event/client.(*CloudEventClient).do
  /workspace/internal/cloud/event/client/client.go:102
gitlab.eng.vmware.com/tanzu-image-signing/amr-observer/internal/cloud/event/client.(*CloudEventClient).checkCloudEventHandlerAvailability.func1
  /workspace/internal/cloud/event/client/client.go:123
github.com/sony/gobreaker.(*CircuitBreaker).Execute
  /go/pkg/mod/github.com/sony/gobreaker@v0.5.0/gobreaker.go:242
gitlab.eng.vmware.com/tanzu-image-signing/amr-observer/internal/cloud/event/client.(*CloudEventClient).checkCloudEventHandlerAvailability
  /workspace/internal/cloud/event/client/client.go:117
gitlab.eng.vmware.com/tanzu-image-signing/amr-observer/internal/cloud/event/client.(*CloudEventClient).CheckAvailabilityPeriodically
  /workspace/internal/cloud/event/client/client.go:111
```

Sample log in AMR CloudEvent Handler:

```json
{"level":"error","ts":"2023-06-20T15:15:04.321672436Z","caller":"amr-persister/main.go:99","msg":"AMR is unavailable: Get \"https://amr-graphql-app.metadata-store.svc.cluster.local:8443/play\": dial tcp 10.28.116.184:8443: connect: connection refused","stacktrace":"<...>"}
```

```json
{"level":"error","ts":"2023-06-20T20:13:15.689628865Z","caller":"amr-persister/main.go:102","msg":"MDS is unavailable: Get \"https://metadata-store-app.metadata-store.svc.cluster.local:8443/api/health\": dial tcp 10.28.122.145:8443: connect: connection refused","stacktrace":"..."}
```

---

An unsupported protocol is used for the `amr.observer.cloudevent_handler.endpoint`, as seen in this
example:

```log
2023-06-28T18:48:31Z  ERROR  httpclient  error sending request to AMR CloudEvent Handler  {"error": "Get \"amr-persister.example.com/healthz\": unsupported protocol scheme \"\""}

...

2023-06-28T18:48:31Z  ERROR  setup  unable to registry location  {"error": "failed to send, Post \"amr-persister.example.com\": unsupported protocol scheme \"\""}

...

2023-06-28T18:48:31Z  ERROR  httpclient.circuitbreaker  error contacting event handler  {"error": "Get \"amr-persister.example.com/healthz\": unsupported protocol scheme \"\""}
```

To fix this, use the necessary `https://` or `http://` prepended protocol.

---

If the log contains:

```log
2023/06/28 19:09:06 No certs appended, using system certs only
2023-06-28T19:09:06Z  INFO  controller-runtime.metrics  Metrics server is starting to listen  {"addr": "127.0.0.1:8080"}
2023-06-28T19:09:06Z  INFO  setup  Establishing  {"locationId": "d9fa1ee9-ba42-4262-aaf6-4128f99096b9"}
```

and if the following description of the pod shows:

```console
$ kubectl -n amr-observer-system describe pods "<amr-observer-controller-manager-...>"
...

Events:
  Type     Reason     Age                    From               Message
  ----     ------     ----                   ----               -------
...

  Warning  Unhealthy  3m35s (x3 over 4m15s)  kubelet            Liveness probe failed: Get "http://192.168.45.78:8081/healthz": context deadline exceeded (Client.Timeout exceeded while awaiting headers)

...

  Warning  Unhealthy  3m5s (x10 over 4m25s)  kubelet            Readiness probe failed: Get "http://192.168.45.78:8081/readyz": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
```

this is a symptom of a wrong protocol for `amr.observer.cloudevent_handler.endpoint`. The fix is to
use the necessary `https://` or `http://` prepended protocol, depending on the TLS configuration.

---

## <a id='observer-logs'></a> AMR Observer Logs

To see the AMR Observer logs by running:

```console
kubectl -n amr-observer-system logs deployments/amr-observer-controller-manager
```

---

If the AMR Observer is not observing the `ImageVulnerabilityScan` CRD, you see:

```log
2023-06-20T15:47:09Z  INFO  ivs.SetupWithManager  Not registering ImageVulnerabilityScans Controller: customresourcedefinitions.apiextensions.k8s.io "imagevulnerabilityscans.app-scanning.apps.tanzu.vmware.com" not found"
```

---

When the AMR Observer is observing the `ImageVulnerabilityScan` CRD, if it is installed, you see:

```log
2023-06-28T17:56:43Z  INFO  Starting Controller  {"controller": "imagevulnerabilityscan", "controllerGroup": "app-scanning.apps.tanzu.vmware.com", "controllerKind": "ImageVulnerabilityScan"}
```

---

When Observer first starts up, it registers a location to the Metadata Store:

```log
2023-06-20T15:47:09Z  INFO  setup  Establishing  {"locationId": "f17f073d-dfec-4624-953e-a133b694ecad"}
sent: type: vmware.tanzu.apps.location.created.v1
  source: f17f073d-dfec-4624-953e-a133b694ecad
  id: 1667244096281455275
Extensions:
map[]
```

For more information, see [Configure Artifact Metadata Repository](configuration.hbs.md).

---

Information about what CA certificates were added to the Observer's HTTP client appears as:

```log
2023-06-20T15:47:05Z  INFO  setup  No additional certs read from configured path, continuing with system truststore"  {"path": "/truststore/ca.crt"}
```

---

No valid CA certificates being found appears as:

```log
2023/06/28 18:41:04 No certs appended, using system certs only
```

---

When the `ReplicaSet` is observed to be deleted, it attempts to send the result to AMR CloudEvent
Handler. There is a known minor issue that even non-workload `ReplicaSet`s are sent during create
and delete events. The log shows an error, however, if it is a no-op because AMR CloudEvent Handler
filters out `ReplicaSet` CloudEvents without a workload container:

```log
result: 204: sent: type: dev.knative.apiserver.resource.delete
  source: f17f073d-dfec-4624-953e-a133b694ecad
2023-06-20T19:01:23Z  INFO  replicaset.sendCloudEvent  received result  {"ceType": "dev.knative.apiserver.resource.delete", "result": "500: "}
  id: 4565357219078868493
Extensions:
map[kind:ReplicaSet name:amr-graphql-app-7ff9bc58f namespace:metadata-store]
```

There is partial legacy support for Knative `ApiServerSource` CloudEvents that the Observer uses,
which is why there are CloudEvents formatted similarly to the Knative `APIServerSource` being sent.

## <a id='cloudevent-logs'></a> AMR CloudEvent Handler Logs

```console
kubectl -n metadata-store logs deployments/amr-persister
```

---

When an event is received, there are log entries for an event being handled:

```json
{"level":"debug","ts":"2023-06-20T15:47:09.203079201Z","caller":"persister/persisteradapter.go:76","msg":"Handle single event"}
{"level":"debug","ts":"2023-06-20T15:47:09.203160964Z","caller":"persister/persisteradapter.go:108","msg":"Handle Event"}
```

---

When the Observer starts up, it sends the Location CloudEvent, and the AMR CloudEvent Handler sends
the location information to the Metadata Store. The log entries contain where the request is sent,
the JSON payload, and the response body returned from the Metadata Store:

```json
{"level":"debug","ts":"2023-06-20T15:47:09.203390152Z","caller":"persister/persisteradapter.go:117","msg":"Registry Location Event"}
{"level":"info","ts":"2023-06-20T15:47:09.20349837Z","caller":"persister/persisteradapter.go:279","msg":"Sending request to: https://amr-graphql-app.metadata-store.svc.cluster.local:8443/api/v1/locations with payload: {\"alias\":\"f17f073d-dfec-4624-953e-a133b694ecad\",\"reference\":\"f17f073d-dfec-4624-953e-a133b694ecad\"}"}
{"level":"info","ts":"2023-06-20T15:47:09.238033374Z","caller":"persister/persisteradapter.go:299","msg":"status: 200, responseBody: {\"alias\":\"f17f073d-dfec-4624-953e-a133b694ecad\",\"reference\":\"f17f073d-dfec-4624-953e-a133b694ecad\",\"labels\":null}\n"}
```

---

When there is an error sending Location to the Metadata Store you see:

```json
{"level":"debug","ts":"2023-06-20T15:15:04.294883639Z","caller":"persister/persisteradapter.go:117","msg":"Registry Location Event"}
{"level":"info","ts":"2023-06-20T15:15:04.295528237Z","caller":"persister/persisteradapter.go:279","msg":"Sending request to: https://amr-graphql-app.metadata-store.svc.cluster.local:8443/api/v1/locations with payload: {\"alias\":\"f17f073d-dfec-4624-953e-a133b694ecad\",\"reference\":\"f17f073d-dfec-4624-953e-a133b694ecad\"}"}
{"level":"error","ts":"2023-06-20T15:15:04.304782986Z","caller":"persister/persisteradapter.go:283","msg":"post error: Post \"https://amr-graphql-app.metadata-store.svc.cluster.local:8443/api/v1/locations\": dial tcp 10.28.116.184:8443: connect: connection refused","stacktrace":"<...>"}
{"level":"error","ts":"2023-06-20T15:15:04.304878714Z","caller":"amr-persister/main.go:72","msg":"Error Handler received error Post \"https://amr-graphql-app.metadata-store.svc.cluster.local:8443/api/v1/locations\": dial tcp 10.28.116.184:8443: connect: connection refused","stacktrace":"..."}
```

---

When the AMR CloudEvent Handler is up and ready to receive CloudEvents you see:

```json
{"level":"info","ts":"2023-06-20T15:14:56.308829574Z","caller":"amr-persister/main.go:61","msg":"Start Receiving"}
```

---

When the Artifact Metadata Repository is unavailable you see:

```json
{"level":"error","ts":"2023-06-20T19:03:47.006718745Z","caller":"amr-persister/main.go:99","msg":"AMR is unavailable: Get \"https://amr-graphql-app.metadata-store.svc.cluster.local:8443/play\": dial tcp 10.28.116.184:8443: connect: connection refused","stacktrace":"..."}
```

---

When the Metadata Store is unavailable you see:

```json
{"level":"error","ts":"2023-06-20T20:13:15.689628865Z","caller":"amr-persister/main.go:102","msg":"MDS is unavailable: Get \"https://metadata-store-app.metadata-store.svc.cluster.local:8443/api/health\": dial tcp 10.28.122.145:8443: connect: connection refused","stacktrace":"..."}
```

---

When the received `ReplicaSet` CloudEvent did not contain a "workload" container you see:

```json
{"level":"error","ts":"2023-06-20T20:13:11.60733257Z","caller":"persister/persisteradapter.go:218","msg":"generate application payload error: unable to find workload container","stacktrace":"..."}
```

AMR only supports `ReplicaSet`s generated during a Workload run and containing a workload named
`container`.

---

The report already exists. This occurs when you reach the resync period and the controllers are
configured to reconcile the `ImageVulnerability` custom resource. This error is not an issue if the
previous submission of the report was successful. For example:

```json
{"level":"info","ts":"2023-06-21T20:54:32.772037121Z","caller":"mds/handle-event.go:107","msg":"Error posting image report: Validation failed: {\"message\":\"a report with uid 'd1625dd5ad94c2c5f83a8de4c3a3c382e4e4774a77c2e71fcf68a0cd3f953f7b' already exists\"}\n"}
{"level":"error","ts":"2023-06-21T20:54:32.772060231Z","caller":"amr-persister/main.go:72","msg":"Error Handler received error Validation failed: {\"message\":\"a report with uid 'd1625dd5ad94c2c5f83a8de4c3a3c382e4e4774a77c2e71fcf68a0cd3f953f7b' already exists\"}\n","stacktrace":"..."}
```
