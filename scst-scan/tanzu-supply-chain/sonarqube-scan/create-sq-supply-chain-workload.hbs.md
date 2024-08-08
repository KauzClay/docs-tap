# Create a workload from the supply chain

This topic tells you how to create, apply, and observe a workload from a supply chain containing the
SonarQube Scan component.

## <a id="prepare"></a> Prepare

To prepare:

- [Install the Tanzu CLI](../../../install-tanzu-cli.hbs.md).
- [Install the Tanzu Supply Chain CLI plug-ins](../../../supply-chain/platform-engineering/how-to/install-the-cli.hbs.md).
- Set up a SonarQube server to upload scan results to. For steps, see the
  [SonarQube documentation](https://docs.sonarsource.com/sonarqube/latest/setup-and-upgrade/install-the-server/introduction/).
- Create a project in the SonarQube server. For steps, see the
  [SonarQube documentation](https://docs.sonarsource.com/sonarqube/latest/project-administration/creating-and-importing-projects/).
- Generate a project token in the SonarQube project. For steps, see the
  [SonarQube documentation](https://docs.sonarsource.com/sonarqube/latest/user-guide/user-account/generating-and-using-tokens/).
- [Create a supply chain that performs a SonarQube scan](create-supply-chain-with-sq.hbs.md).

## <a id="define-and-create-wl"></a> Define and create a workload

This section tells you how to create a workload from an existing supply chain that contains the
SonarQube Scan component. For more information, see
[Work with workloads](../../../supply-chain/development/how-to/discover-workloads.hbs.md).

You can define a workload directly in a YAML file or use the Tanzu Workload CLI plug-in to generate
a workload manifest.

Define a workload YAML file
: To define `workload.yaml`:

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
         sonar-project-name: SONAR-PROJECT-NAME
         sonar-project-key: SONAR-PROJECT-KEY
         sonar-token-secret-name: SONAR-TOKEN-SECRET-NAME
         sonar-project-base-dir: SONAR-PROJECT-BASE-DIR
         project-type: PROJECT-TYPE
         jdk-url: JDK-URL
         debug-mode: DEBUG-MODE
    ```

  Where:

  - `KIND` is the `kind` defined in the SonarQube supply chain. The kind can also be found in the
    `supplychain` YAML generated in the `supplychains` directory.
  - `API-VERSION` is defined in the SonarQube Supply Chain. `API-VERSION` is `<group>`/`<version>`
    found in the `supplychain` YAML in the `supplychains` directory.
  - `WORKLOAD-NAME` is your workload name.
  - `GIT-URL` is the Git repository URL to clone from for the source component.
  - `GIT-BRANCH` is the Git branch reference to watch for the new source.
  - `GIT-SUBPATH` is the path inside the bundle to locate source code.
  - `SONAR-PROJECT-NAME` is the display name of the project being scanned in the SonarQube server.
  - `SONAR-PROJECT-KEY` is the project key for the project in SonarQube. It is optional and is the
    same as `sonar-project-name` if unset.
  - `SONAR-TOKEN-SECRET-NAME` is the name of the Kubernetes secret that contains the SonarQube
    project token generated in the SonarQube server. The secret must store the token under the key
    `sonar-token`. For more information about generating the token, see the
    [SonarQube documentation](https://docs.sonarsource.com/sonarqube/latest/user-guide/user-account/generating-and-using-tokens/).
  - `SONAR-PROJECT-BASE-DIR` is the path to the directory to scan from the source code root. It is
    empty by default.
  - `PROJECT-TYPE` is either `maven` or `gradle` depending on the source code project. Only Maven
    and Gradle projects are currently supported.
  - `JDK-URL` is the URL to download the JDK version that is compatible with the source project.
    If not given, the default JDK version installed in the task image is used.
  - `DEBUG-MODE` is for enabling debug logs in the scan. It expects `true` or `false`. It is `false`
    by default.

  For more information about any of the `GIT-*` values, see
  [Source Git Provider](../../../supply-chain/reference/catalog/about.hbs.md#source-git-provider).

Generate a workload manifest
: To render sample workload YAML that you can configure and put in `workload.yaml`:

  1. Use the Tanzu Workload CLI plug-in to see available `Workload` kinds by running:

     ```console
     tanzu workload kinds list
     ```

  1. Record the listed name of the kind, as defined in the
     [Create a supply chain with the SonarQube Scan component](create-supply-chain-with-sq.hbs.md#sonarqube-scan)
     section:

  1. Pass the `kind` into the Tanzu Workload CLI plug-in to generate a workload by running:

     ```console
     tanzu workload generate NAME --kind KIND
     ```

     Where `KIND` is your kind.

## <a id="create-workload"></a> Create a workload

Create the workload by running:

```console
tanzu workload create --file workload.yaml --namespace DEV-NAMESPACE
```

Where `DEV-NAMESPACE` is the developer namespace.

## <a id="observe-workload"></a> Observe the workload

To observe the workload:

1. See the progress of the workload running through the supply chain by running:

   ```console
   watch -n 0.5 kubectl get pipelineruns,taskruns,pods, supplychain -n DEV-NAMESPACE
   ```

   Where `DEV-NAMESPACE` is the developer namespace.

1. See the detailed output of the supply chain by running:

   ```console
   tanzu workload run get WORKLOAD-RUN-NAME -n DEV-NAMESPACE --show-details
   ```

For more information, see
[Work with workloads](../../../supply-chain/development/how-to/discover-workloads.hbs.md).
