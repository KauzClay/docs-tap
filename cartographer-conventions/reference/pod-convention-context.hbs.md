# PodConventionContext for Cartographer Conventions

This reference topic describes the `PodConventionContext` that you can use with Cartographer
Conventions.

## Overview

The Pod convention context is the body of the webhook request and response. The convention
controller provides the specification. The convention server sets the status.

The context consists of:

- A wrapper of the individual object description in an API (TypeMeta)
- The persistent metadata of a resource
  ([ObjectMeta](https://kubernetes.io/docs/reference/kubernetes-api/common-definitions/object-meta/#ObjectMeta))
- The [PodConventionContextSpec](pod-convention-context-spec.hbs.md)
- The [PodConventionContextStatus](pod-convention-context-status.hbs.md)

## PodConventionContext Objects

In the `PodConventionContext` API resource:

- The object path `.spec.template` field defines the `PodTemplateSpec` to be enriched by
  conventions. For more information about `PodTemplateSpec`, see the
  [Kubernetes documentation](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-template-v1/#PodTemplateSpec).
- The object path `.spec.imageConfig[]` field defines [ImageConfig](image-config.hbs.md). Each entry
  is populated with the name of the image (`.spec.imageConfig[].image`) and its OCI metadata
  (`.spec.imageConfig[].config`). These entries are generated for each image referenced in
  [PodTemplateSpec](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-template-v1/#PodTemplateSpec)
  (`.spec.template`).

The following snippet is an example of a `PodConventionContext` resource request that the convention
server receives. This resource is generated for a
[Go language-based application image](https://github.com/paketo-buildpacks/samples/tree/main/go/mod)
in GitHub. It is built with Cloud Native Paketo Buildpacks that use Go mod for dependency
management.

```yaml
---
apiVersion: webhooks.conventions.carto.run/v1alpha1
kind: PodConventionContext
metadata:
  name: sample # the name of the ClusterPodConvention
spec: # the request
  imageConfig: # one entry per image referenced by the PodTemplateSpec
  - image: sample/go-based-image
    boms:
    - name: cnb-app:.../sbom.cdx.json
      raw: ...
    config:
      entrypoint:
      - "/cnb/process/web"
      domainname: ""
      architecture: "amd64"
      image: "sha256:05b698a4949db54fdb36ea431477867abf51054abd0cbfcfd1bb81cda1842288"
      labels:
        "io.buildpacks.stack.distro.version": "18.04"
        "io.buildpacks.stack.homepage": "https://github.com/paketo-buildpacks/stacks"
        "io.buildpacks.stack.id": "io.buildpacks.stacks.bionic"
        "io.buildpacks.stack.maintainer": "Paketo Buildpacks"
        "io.buildpacks.stack.distro.name": "Ubuntu"
        "io.buildpacks.stack.metadata": `{"app":[{"sha":"sha256:ea4ec23266a3af1204fd643de0f3572dd8dbb5697a5ef15bdae844777c19bf8f"}],
        "buildpacks":[{"key":"paketo-buildpac`...,
        "io.buildpacks.build.metadata": `{"bom":[{"name":"go","metadata":{"licenses":[],"name":"Go","sha256":"7fef8ba6a0786143efcce66b0bbfbfbab02afeef522b4e09833c5b550d7`...
  template:
    spec:
      containers:
      - name : workload
        image: helloworld-go-mod
```

## <a id="pcc-structure"></a> PodConventionContext Structure

This section introduces more information about the image configuration in `PodConventionContext`.
The convention controller passes this information for each image in good faith. The convention
controller is not the source of the metadata, and there is no guarantee that the information is
correct.

The `config` field in the image configuration passes through the
[OCI Image metadata in GitHub](https://github.com/opencontainers/image-spec/blob/main/config.md)
loaded from the registry for the image.

The `boms` field in the image configuration passes through the [BOM](bom.hbs.md)s of the image.
Conventions might parse the BOMs they want to inspect. There is no guarantee that an image contains
a BOM or that the BOM is in a specific format.
