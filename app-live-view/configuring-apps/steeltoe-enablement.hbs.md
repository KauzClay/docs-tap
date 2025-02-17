# Enabling Steeltoe apps for Application Live View

This topic describes how developers configure a Steeltoe app to be observed by
Application Live View within Tanzu Application Platform.

Application Live View supports Steeltoe .NET apps with .NET core runtime version `v6.0.8`.

## Enable Steeltoe apps

You can enable Application Live View to interact with a Steeltoe app within Tanzu Application Platform.

To expose management actuator endpoints, add following configuration to your `appsettings.json` file:

```json
{
  "Management": {
    "Endpoints": {
      "Actuator":{
        "Exposure": {
          "Include": [ "*" ]
        }
      }
    }
  }
}
```

To enable logging, add the following configuration to your `appsettings.json` file:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Steeltoe": "Warning",
      "Sample": "Information"
    }
  }
}
```

To enable heapdump, add the following configuration to your `appsettings.json` file:

```json
{
  "Management": {
    "Endpoints": {
      "HeapDump": {
        "HeapDumpType": "Normal"
      }
    }
  }
}
```


Thread metrics is available in SteeltoeVersion `3.2.*`. To enable the Threads page in the Application Live View UI, add the following configuration to your `.csproj` file:

```xml
<PropertyGroup>
    <SteeltoeVersion>3.2.*</SteeltoeVersion>
</PropertyGroup>
```

To enable Application Live View on the Steeltoe TAP workload, the Application Live View convention service automatically applies labels on the workload, such as `tanzu.app.live.view.application.flavours: steeltoe` and `tanzu.app.live.view: true`, based on the Steeltoe image metadata.

Here's an example of creating a workload for a Steeltoe Application:

```console
tanzu apps workload create steeltoe-app --type web --git-repo https://github.com/sample-accelerators/steeltoe-weatherforecast --git-branch main --annotation autoscaling.knative.dev/min-scale=1 --yes --label app.kubernetes.io/part-of=sample-app
```

If your application image is NOT built with Tanzu Build Service, to enable Application Live View on Steeltoe Tanzu Application Platform workload, use the following command. For example:

```
tanzu apps workload create steeltoe-app --type web --app steeltoe-app --image <IMAGE NAME> --annotation autoscaling.knative.dev/min-scale=1 --yes --label tanzu.app.live.view=true --label tanzu.app.live.view.application.name=steeltoe-app --label tanzu.app.live.view.application.flavours=steeltoe
```
