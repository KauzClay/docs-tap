# Catalog of Tanzu Supply Chain Components

{{> 'partials/supply-chain/beta-banner' }}

This section introduces the catalog of components shipped with TAP. You will find all of these
components in the "authoring" profile.

## app-config-server

Version: 1.0.0

### Description:

Generates configuration for a Server application from a Conventions PodIntent.
Server applications contain a K8s Deployment and Service and can be configured with Ingress.

### Inputs

| Name | Type |
| --- | --- |
| conventions | [conventions](./output-types.hbs.md#conventions) |

### Outputs

| Name | Type |
| --- | --- |
| oci-yaml-files | [oci-yaml-files](./output-types.hbs.md#oci-yaml-files) |
| oci-ytt-files | [oci-ytt-files](./output-types.hbs.md#oci-ytt-files) |

### Config

```yaml
spec:
  # Configuration for the registry to use
  registry:
    # The name of the registry server, e.g. docker.io
    # +required
    server:
    # The name of the repository
    # +required
    repository:
```

---

## app-config-web

Version: 1.0.0

### Description:

Generates configuration for a Web application from a Conventions PodIntent.
Web applications contain a Knative Service.

### Inputs

| Name | Type |
| --- | --- |
| conventions | [conventions](./output-types.hbs.md#conventions) |

### Outputs

| Name | Type |
| --- | --- |
| oci-yaml-files | [oci-yaml-files](./output-types.hbs.md#oci-yaml-files) |
| oci-ytt-files | [oci-ytt-files](./output-types.hbs.md#oci-ytt-files) |

### Config

```yaml
spec:
  # Configuration for the registry to use
  registry:
    # The name of the repository
    # +required
    repository:
    # The name of the registry server, e.g. docker.io
    # +required
    server:
```

---

## app-config-worker

Version: 1.0.0

### Description

Generates configuration for a Worker application from a Conventions PodIntent.
Worker applications contain a K8s Deployment.

### Inputs

| Name | Type |
| --- | --- |
| conventions | [conventions](./output-types.hbs.md#conventions) |

### Outputs

| Name | Type |
| --- | --- |
| oci-yaml-files | [oci-yaml-files](./output-types.hbs.md#oci-yaml-files) |
| oci-ytt-files | [oci-ytt-files](./output-types.hbs.md#oci-ytt-files) |

### Config

```yaml
spec:
  # Configuration for the registry to use
  registry:
    # The name of the repository
    # +required
    repository:
    # The name of the registry server, e.g. docker.io
    # +required
    server:
```

---

## buildpack-build

Version: 1.0.0

### Description

Builds an app with buildpacks using kpack

### Inputs

| Name | Type |
| --- | --- |
| source | [source](./output-types.hbs.md#source) |
| git | [git](./output-types.hbs.md#git) |

### Outputs

| Name | Type |
| --- | --- |
| image | [image](./output-types.hbs.md#image) |

### Config

```yaml
spec:
  # Registry to use
  registry:
    # The registry address
    # +required
    server:
    # The repository to use
    # +required
    repository:
  # Kpack build specification
  build:
    # Service account to use
    serviceAccountName:
    env:
    # Configure workload to use a non-default builder or clusterbuilder
    builder:
      # builder kind
      kind:
      # builder name
      name:
    # cache options
    cache:
      # whether to use a cache image
      enabled:
      # cache image to use
      image:
  source:
    # path inside the source to build from (build has no access to paths above the subPath)
    subPath:
```

---

## carvel-package

Version: 1.0.0

### Description

Generates a carvel package from OCI images containing raw YAML files and YTT files.

### Inputs

| Name | Type |
| --- | --- |
| oci-yaml-files | [oci-yaml-files](./output-types.hbs.md#oci-yaml-files) |
| oci-ytt-files | [oci-ytt-files](./output-types.hbs.md#oci-ytt-files) |

### Outputs

| Name | Type |
| --- | --- |
| package | [package](./output-types.hbs.md#package) |

### Config

```yaml
spec:
  # Configuration for the generated Carvel Package
  carvel:
    # The name of the Carvel Package. Combines with spec.carvel.packageDomain to create the Package refName. If set to "", will use the workload name.
    packageName:
    # Service account that gives kapp-controller privileges to create resources in the namespace.
    serviceAccountName:
    # Name of the values Secret that provides customized values to the package installation's templating steps.
    valuesSecretName:
    # PEM encoded certificate data for the image registry where the files will be pushed to.
    caCertData:
    # Enable the use of IAAS based authentication for imgpkg.
    iaasAuthEnabled:
    # The domain of the Carvel Package. Combines with spec.carvel.packageName to create the Package refName. If set to "", will use "default.tap".
    packageDomain:
  gitOps:
    # the branch to commit changes to
    branch:
    # the relative path within the gitops repository to add the package configuration to.
    subPath:
    # the repository to push the pull request to
    url:
  # Configuration for the registry to use
  registry:
    # The name of the repository
    # +required
    repository:
    # The name of the registry server, e.g. docker.io
    # +required
    server:
```

---

## conventions

Version: 1.0.0

### Description

The Conventions component analyzes the `image` input as described in the
[Cartographer Conventions](https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.8/tap/cartographer-conventions-about.html)
documentation and produces a `conventions` output image.

Depends on:

- Managed Resource Controller.
  - Tanzu Carvel Package: `managed-resource-controller.apps.tanzu.vmware.com @ >=0.1.2`
- Conventions Controller
  - Tanzu Carvel Package: `cartographer.tanzu.vmware.com @ >= 0.8.10`

### Inputs

| Name | Type |
| --- | --- |
| image | [image](./output-types.hbs.md#image) |

### Outputs

| Name | Type |
| --- | --- |
| conventions | [conventions](./output-types.hbs.md#conventions) |

### Config

```yaml
spec:
  # May contain an optional array of objects. Each object is a pair of keys: `name` and either `value` or `valueFrom`.
  # The Conventions component will translate these values into environment variables in the output object.
  env:
```

---

## deployer

Version: 1.0.0

### Description

Deploys K8s resources to the cluster.

### Inputs

| Name | Type |
| --- | --- |
| package | [package](./output-types.hbs.md#package) |

### Outputs

* _none_

### Config

```yaml
spec:
  # The path to the yaml to be applied to the cluster.
  subPath:
    # The path to the yaml to be applied to the cluster
    # +required
    path:
```

---

## git-writer

Version: 1.0.0

### Description

Writes carvel package config directly to a gitops repository

### Inputs

| Name | Type |
| --- | --- |
| package | [package](./output-types.hbs.md#package) |

### Outputs

* _none_

### Config

```yaml
spec:
  gitOps:
    # the repository to push the pull request to
    # +required
    url:
    # the branch to commit changes to
    branch:
    # the relative path within the gitops repository to add the package configuration to.
    subPath:
```

---

## git-writer-pr

Version: 1.0.0

### Description

Writes carvel package config to a gitops repository and opens a PR

### Inputs

| Name | Type |
| --- | --- |
| package | [package](./output-types.hbs.md#package) |

### Outputs

| Name | Type |
| --- | --- |
| git-pr | [git-pr](./output-types.hbs.md#git-pr) |

### Config

```yaml
spec:
  gitOps:
    # the base branch to create PRs against
    baseBranch:
    # the relative path within the gitops repository to add the package configuration to.
    subPath:
    # the repository to push the pull request to
    # +required
    url:
```

---

## kaniko-build

Version: 1.0.0

### Description

Builds an app with kaniko

### Inputs

| Name   | Type                                 |
|--------|--------------------------------------|
| source | [source](output-types.hbs.md#source) |
| git    | [git](output-types.hbs.md#git)       |

### Outputs

| Name  | Type                               |
|-------|------------------------------------|
| image | [image](output-types.hbs.md#image) |

### Config

```yaml
spec:
  # Kaniko build specification
  build:
    # path to dockerfile to build
    dockerfile:
    # extra args to pass to kaniko build
    extra-args:
  # Registry to use
  registry:
    # The repository to use
    # +required
    repository:
    # The registry address
    # +required
    server:
```

---

## sonarqube-sast-scan

Version: 1.0.0

### Description

The SonarQube Supply Chain component performs a Static Application Security Testing (SAST) scan by
using the Maven or Gradle Sonar plug-in against the source input.

### Inputs

| Name   | Type                                   |
|--------|----------------------------------------|
| source | [source](./output-types.hbs.md#source) |

### Outputs

* _none_

### Config

```yaml
spec:
  sonarqube:
    # This is the URL of the sonar server.
    # +required
    sonar-host-url:
    # This is the path to the directory to scan from the repository root.
    sonar-project-base-dir:
    # This is the project key for the sonar project. If not set it will be the same as the project name.
    sonar-project-key:
    # This is the display name of the project in the sonar server.
    # +required
    sonar-project-name:
    # This is the name of the secret that contains the Sonarqube project token. See the Sonarqube documentation for more details: https://docs.sonarsource.com/sonarqube/latest/user-guide/user-account/generating-and-using-tokens/.
    # +required
    sonar-token-secret-name:
    # This is the project type of source, only "maven" or "gradle" are supported.
    # +required
    project-type:
    # This is the url to download the jdk version compatible with the source project. If not given, the default jdk version installed in the task image will be used.
    jdk-url:
    # This is for enabling debug logs in the scan, expects "true" or "false". It is false by default.
    debug-mode:
```

---

## source-git-provider

Version: 1.0.0

### Description

Source git provider retrieves source code and monitors a git repository.

### Inputs

* _none_

### Outputs

| Name | Type |
| --- | --- |
| source | [source](./output-types.hbs.md#source) |
| git | [git](./output-types.hbs.md#git) |

### Config

```yaml
spec:
  source:
    # Use this object to retrieve source from a git repository.
    # The tag, commit and branch fields are mutually exclusive, use only one.
    # +required
    git:
      # A git branch ref to watch for new source
      branch:
      # A git commit sha to use
      commit:
      # A git tag ref to watch for new source
      tag:
      # The url to the git source repository
      # +required
      url:
    # The sub path in the bundle to locate source code
    subPath:
```

---

## source-package-translator

Version: 1.0.0

### Description

Takes the type source and immediately outputs it as type package.

### Inputs

| Name | Type |
| --- | --- |
| source | [source](./output-types.hbs.md#source) |

### Outputs

| Name | Type |
| --- | --- |
| package | [package](./output-types.hbs.md#package) |

### Config

_none_

---

## trivy-image-scan

Version: 1.0.0

### Description

Performs a trivy image scan using the scan 2.0 components

### Inputs

| Name | Type |
| --- | --- |
| image | [image](./output-types.hbs.md#image) |
| git | [git](./output-types.hbs.md#git) |

### Outputs

* _none_

### Config

```yaml
spec:
  # Configuration for the registry to use
  registry:
    # The name of the repository
    # +required
    repository:
    # The name of the registry server, e.g. docker.io
    # +required
    server:
  source:
    # Fill this object in if you want your source to come from git.
    # The tag, commit and branch fields are mutually exclusive, use only one.
    # +required
    git:
      # A git branch ref to watch for new source
      branch:
      # A git commit sha to use
      commit:
      # A git tag ref to watch for new source
      tag:
      # The url to the git source repository
      # +required
      url:
    # The sub path in the bundle to locate source code
    subPath:
  # Image Scanning configuration
  scanning:
    service-account-scanner:
    workspace:
      size:
      bindings:
    active-keychains:
    service-account-publisher:
```

---

## sonarqube-sast-scan

Version: 1.0.0

### Description

Performs a SonarQube sast scan

### Inputs

| Name | Type |
| --- | --- |
| git | [git](./output-types.hbs.md#git) |

### Outputs

* _none_

### Config

```yaml
spec:
  source:
    # This is used to retrieve source from a git repository.
    # The tag, commit and branch fields are mutually exclusive, use only one.
    # +required
    git:
      # A git branch ref to watch for new source
      branch:
      # A git commit sha to use
      commit:
      # A git tag ref to watch for new source
      tag:
      # The url to the git source repository
      # +required
      url:
    # The sub path in the bundle to locate source code
    subPath:
  # SonarQube Scan configuration
  sonarqube:
    # SonarQube server url
    # +required
    sonar-host-url:
    # The project display name in the SonarQube server
    # +required
    sonar-project-name:
    # The project key defined in the SonarQube server
    sonar-project-key:
    # The name of the secret that contains the SonarQube project token
    # +required
    sonar-token-secret-name:
    # Path to the directory to scan from the source code root
    sonar-project-base-dir:
    # The project type of source, only "maven" or "gradle" are supported
    # +required
    project-type:
    # The url to download the jdk version compatible with the source project
    jdk-url:
    # Enable debug logs in the scan, expects "true" or "false"
    debug-mode:
```
