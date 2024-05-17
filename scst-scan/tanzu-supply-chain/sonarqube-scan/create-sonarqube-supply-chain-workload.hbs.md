# Create a Workload from the Supply Chain

This topic describes how to create, apply, and observe a workload from a Tanzu Supply Chain containing the SonarQube scan component.

## <a id="prerequisites"></a> Prerequisites

Tanzu CLI plug-ins

- Tanzu Workload CLI plug-in , see [Tanzu Supply Chain CLI plug-ins](../../supply-chain/platform-engineering/how-to/install-the-cli.hbs.md).

SonarQube Server

- Running SonarQube server to upload scan results to, see [SonarQube installation](https://docs.sonarsource.com/sonarqube/latest/setup-and-upgrade/install-the-server/introduction/)
- Created project in the SonarQube server, see [SonarQube creating project](https://docs.sonarsource.com/sonarqube/latest/project-administration/creating-and-importing-projects/)
- Generated project token in the SonarQube project, see [SonarQube generating tokens](https://docs.sonarsource.com/sonarqube/latest/user-guide/user-account/generating-and-using-tokens/)

Supply Chain with SonarQube scan
- Created [SonarQube Supply Chain](create-supply-chain-with-sonarqube.hbs.md)

## <a id="define-and-create-wl"></a> Define and create a workload

This section tells you how to create a workload from an existing supply chain that contains the SonarQube scan component.

For more information about how to create a workload, see [Work with Workloads](../../supply-chain/development/how-to/discover-workloads.hbs.md).

You can define a workload in `yaml` file or use the Tanzu Workload CLI plug-in to generate a workload manifest.

### <a id="define-workload"></a> Define a workload

Define a `workload.yaml`:

```yaml
kind: KIND
apiVersion: API-VERSION
metadata:
  name: WORKLOAD-NAME
spec:
  source:
    git:
      branch: GIT-BRANCH
      url: GIT-URL
    subPath: GIT-SUBPATH
  sonarqube:
     sonar-host-url: SONAR-URL
     sonar-project-name: SONAR-PROJECT-NAME
     sonar-project-key: SONAR-PROJECT-KEY
     sonar-token: SONAR-TOKEN
     sonar-project-base-dir: SONAR-PROJECT-BASE-DIR
```

Where:

- `KIND` is the kind defined in the [SonarQube Supply Chain](create-supply-chain-with-sonarqube.hbs.md#sonarqube-scan). The kind can also be found in the supplychain yaml generated in the supplychains directory.
- `API-VERSION` is defined in the [SonarQube Supply Chain](create-supply-chain-with-sonarqube.hbs.md#sonarqube-scan). `API-VERSION` is `<group>`/`<version>` found in the supplychain yaml in the supplychains directory.
- `WORKLOAD-NAME` is your workload name.
- `GIT-URL` is the Git repository URL to clone from for the source component.
- `GIT-BRANCH` is the Git branch ref to watch for the new source.
- `GIT-SUBPATH` is the path inside the bundle to locate source code.
- `SONAR-URL` is the url to the SonarQube server to upload scan results to.
- `SONAR-PROJECT-NAME` is the display name of the project being scanned in the SonarQube server.
- `SONAR-PROJECT-KEY` is the project key for the project in SonarQube. It is optional and is the same as `sonar-project-name` if unset. 
- `SONAR-TOKEN` is the SonarQube project token generate in the SonarQube server, see [SonarQube docs](https://docs.sonarsource.com/sonarqube/latest/user-guide/user-account/generating-and-using-tokens/) for more detail.
- `SONAR-PROJECT-BASE-DIR` is the path to the directory to scan from the source code root. It is empty by default.

For more information about any of the `GIT-*` values, see [Source Git Provider](../../supply-chain/reference/catalog/about.hbs.md#source-git-provider).

### <a id="generate-workload"></a> Generate a workload

1. Use the Tanzu Workload CLI plug-in to see available `Workload` kinds. Note the listed name of the kind defined in [SonarQube Supply Chain](create-supply-chain-with-sonarqube.hbs.md#sonarqube-scan):

```console
tanzu workload kinds list
```

1. Pass the kind into the Tanzu Workload CLI plug-in to generate a workload:

```console
tanzu workload generate NAME --kind KIND
```

This renders a sample workload YAML that you can configure and put in a `workload.yaml`.

### <a id="create-workload"></a> Create a workload

Using the `workload.yaml` created in the previous section, create the workload:

```console
tanzu workload create --file workload.yaml --namespace DEV-NAMESPACE
```

## <a id="observe-workload"></a> Observe workload

Observe the progress of the workload running through the supply chain:

```console
watch -n 0.5 kubectl get pipelineruns,taskruns,pods, supplychain -n DEV-NAMESPACE
```

For more information, [How to observe the Runs of your Workload](../../supply-chain/development/how-to/discover-workloads.hbs.md).
