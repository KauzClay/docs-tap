# Prerequisites for Prisma Scanner for Supply Chain Security Tools - Scan (Alpha)

This topic describes prerequisites you must complete to install SCST - Scan (Prisma) from the VMware
package repository.

> **Caution** This integration is in Alpha, which means that it is still in active development by
> the Tanzu Practices Global Tech Team and might be subject to change at any point. You might
> encounter unexpected behavior.

## Verify the latest alpha package version

Run this command to output a list of available tags:

```console
imgpkg tag list -i projects.registry.vmware.com/tanzu_practice/tap-scanners-package/prisma-repo-scanning-bundle | sort -V
```

Use the latest version returned in place of the sample version in this topic. For example,
`0.1.5-alpha.13` in the following output.

```console
imgpkg tag list -i projects.registry.vmware.com/tanzu_practice/tap-scanners-package/prisma-repo-scanning-bundle | sort -V
0.1.4-alpha.11
0.1.4-alpha.12
0.1.4-alpha.15
0.1.5-alpha.11
0.1.5-alpha.12
0.1.5-alpha.13
```

## Relocate images to a registry

You must relocate the images from `tanzu.packages.broadcom.com` to your own container
image registry before installing.

{{> 'partials/scst-scan/scan-1-0-deprecation' }}

The Prisma Scanner is in the Alpha development phase, and not packaged as part of Tanzu Application
Platform. It is hosted on `tanzu.packages.broadcom.com`.

For information about supported registries, see each registry's documentation.

To relocate images from `tanzu.packages.broadcom.com` to your registry:

1. Retrieve your Broadcom registry API token:

    1. Sign in to the [Broadcom Support Portal](https://support.broadcom.com).

    1. Go to [Tanzu Application Platform (TAP)](https://support.broadcom.com/group/ecx/productdownloads?subfamily=VMware%20Tanzu%20Application%20Platform)
       and expand the **VMware Tanzu Application Platform** dropdown.

    1. Click the Token Download icon next to the Tanzu Application Platform version you want to
       download.

        ![Screenshot of the Tanzu Application Platform download page in the Broadcom Support Portal
          with the Token Download icon highlighted.](../images/download-token-icon.png)

    1. Follow the instructions in the dialog box. Save the token as a variable named
       `MY_BROADCOM_SUPPORT_ACCESS_TOKEN`. For example:

        ```console
        export MY_BROADCOM_SUPPORT_ACCESS_TOKEN=API-TOKEN
        ```

        Where `API-TOKEN` is your token from the Broadcom Support Portal.

1. Set up environment variables for installation:

   ```console
   export IMGPKG_REGISTRY_HOSTNAME_0=tanzu.packages.broadcom.com
   export IMGPKG_REGISTRY_USERNAME_0=MY-BROADCOM-SUPPORT-USERNAME
   export IMGPKG_REGISTRY_PASSWORD_0=${MY_BROADCOM_SUPPORT_ACCESS_TOKEN}
   export INSTALL_REGISTRY_HOSTNAME=MY-REGISTRY
   export INSTALL_REGISTRY_USERNAME=MY-REGISTRY-USER
   export INSTALL_REGISTRY_PASSWORD=MY-REGISTRY-PASSWORD
   export VERSION=VERSION-NUMBER
   export INSTALL_REPO=TARGET-REPOSITORY
   ```

   Where:

   - `MY-BROADCOM-SUPPORT-USERNAME` is the user with access to the images in `tanzu.packages.broadcom.com`.
   - `MY-REGISTRY` is your own registry.   
   - `MY-REGISTRY-USER` is the user with write access to `MY-REGISTRY`.
   - `MY-REGISTRY-PASSWORD` is the password for `MY-REGISTRY-USER`.
   - `VERSION-NUMBER` is your Prisma Scanner version. For example, `0.1.4-alpha.12`.
   - `TARGET-REPOSITORY` is your target repository, a directory or repository on `MY-REGISTRY` that
     serves as the location for the installation files for Prisma Scanner.

2. Install the Carvel tool imgpkg CLI.
   See [Deploying Cluster Essentials](https://docs.vmware.com/en/Cluster-Essentials-for-VMware-Tanzu/{{ vars.ce_version }}/cluster-essentials/deploy.html#optionally-install-clis-onto-your-path-6).

3. Relocate images with the imgpkg CLI by running:

   ```console
   imgpkg copy -b projects.registry.vmware.com/tanzu_practice/tap-scanners-package/prisma-repo-scanning-bundle:${VERSION} \
   --to-repo ${INSTALL_REGISTRY_HOSTNAME}/${INSTALL_REPO}/prisma-repo-scanning-bundle
   ```

## Add the Prisma Scanner package repository

Tanzu CLI packages are available on repositories. Adding the Prisma Scanning package repository
makes the Prisma Scanning bundle and its packages available for installation.

VMware recommends installing the Prisma Scanner objects in the existing `tap-install` namespace to
keep the Prisma Scanner grouped logically with the other Tanzu Application Platform components.

1. Add the Prisma Scanner package repository to the cluster by running:

   ```console
   tanzu package repository add prisma-scanner-repository \
     --url ${INSTALL_REGISTRY_HOSTNAME}/${INSTALL_REPO}/prisma-repo-scanning-bundle:$VERSION \
     --namespace tap-install
   ```

2. Get the status of the Prisma Scanner package repository, and ensure that the status updates to
   `Reconcile succeeded` by running:

   ```console
   tanzu package repository get prisma-scanner-repository --namespace tap-install
   ```

   For example:

   ```console
   $ tanzu package repository get prisma-scanning-repository --namespace tap-install
   - Retrieving repository prisma-scanner-repository...
   NAME:          prisma-scanner-repository
   VERSION:       71091125
   REPOSITORY:    projects.registry.vmware.com/tanzu_practice/tap-scanners-package/prisma-repo-scanning-bundle
   TAG:           0.1.4-alpha.12
   STATUS:        Reconcile succeeded
   REASON:
   ```

3. List the available packages by running:

   ```console
   tanzu package available list --namespace tap-install
   ```

   For example:

   ```console
   $ tanzu package available list --namespace tap-install
   / Retrieving available packages...
     NAME                                                 DISPLAY-NAME                                                              SHORT-DESCRIPTION
     prisma.scanning.apps.tanzu.vmware.com                Prisma for Supply Chain Security Tools - Scan                             Default scan templates using Prisma
   ```

## Prepare the Prisma Scanner configuration

Before installing the Prisma scanner, you must create the configuration and a Kubernetes secret that
contains credentials to access Prisma Cloud.

### Obtain Console URL and Access Keys and Token

The Prisma Scanner supports two methods of authentication:

- Basic authentication with API Key and Secret
- Token-based Authentication

The steps to configure both are outlined to allow you to decide which option you use.

> **Note** The token method issued by Prisma Cloud has a expiration of 1 hour, so it requires
> frequent refreshing.

To obtain your Prisma Compute Console URL and Access Keys and Token.
See [Access keys](https://docs.paloaltonetworks.com/prisma/prisma-cloud/prisma-cloud-admin-compute/authentication/access_keys)
in the Palo Alto Networks documentation.

> **Note** Generated tokens expire after an hour.

#### Access key and secret authentication

To create a Prisma secret, use the following instructions.

1. Create a Prisma secret YAML file and insert the base64 encoded Prisma API token into the `prisma_token`:

    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: PRISMA-ACCESS-KEY-SECRET
      namespace: APP-NAME
    data:
      username: BASE64-PRISMA-ACCESS-KEY-ID
      password: BASE64-PRISMA-ACCESS-KEY-PASSWORD
    ```

   Where:

   - `PRISMA-ACCESS-KEY-SECRET` is the name of your Prisma token secret.
   - `APP-NAME` is the namespace you want to use.
   - `BASE64-PRISMA-ACCESS-KEY-ID` is your base64 encoded Prisma Access Key ID.
   - `BASE64-PRISMA-ACCESS-KEY-PASSWORD` is your base64 encoded Prisma Access Key Password.

2. Apply the Prisma secret YAML file by running:

   ```console
   kubectl apply -f YAML-FILE
   ```

   Where `YAML-FILE` is the name of the Prisma secret YAML file you created.

3. Define the `--values-file` flag to customize the default configuration. You must define the
   following fields in the `values.yaml` file for the Prisma Scanner configuration. You can add
   fields to activate or deactivate behaviors. You can append the values to this file as shown later
   in this topic. Create a `values.yaml` file by using the following configuration:

    ```yaml
    ---
    namespace: DEV-NAMESPACE
    targetImagePullSecret: TARGET-REGISTRY-CREDENTIALS-SECRET
    prisma:
      url: PRISMA-URL
      basicAuth:
        name: PRISMA-ACCESS-KEY-SECRET
    ```

   Where:

   - `DEV-NAMESPACE` is your developer namespace. To use a namespace other than the default
     namespace, ensure that the namespace exists before you install. If the namespace does not
     exist, the scanner installation fails.
   - `TARGET-REGISTRY-CREDENTIALS-SECRET` is the name of the secret that contains the
     credentials to pull an image from a private registry for scanning.
   - `PRISMA-URL` is the FQDN of your Twistlock server.
   - `PRISMA-CONFIG-SECRET` is the name of the secret you created that contains the
     Prisma configuration to connect to Prisma. This field is required.

The Prisma integration can work with or without the SCST - Store integration. The `values.yaml` file
is slightly different for each configuration.

#### Access Token Authentication

1. Create a Prisma secret YAML file and insert the base64 encoded Prisma API token into the `prisma_token`:

    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: PRISMA-TOKEN-SECRET
      namespace: APP-NAME
    data:
      prisma_token: BASE64-PRISMA-API-TOKEN
    ```

   Where:

   - `PRISMA-TOKEN-SECRET` is the name of your Prisma token secret.
   - `APP-NAME` is the namespace you want to use.
   - `BASE64-PRISMA-API-TOKEN` is the name of your base64 encoded Prisma API token.

2. Apply the Prisma secret YAML file by running:

   ```console
   kubectl apply -f YAML-FILE
   ```

   Where `YAML-FILE` is the name of the Prisma secret YAML file you created.

3. Define the `--values-file` flag to customize the default configuration. You must define the
   following fields in the `values.yaml` file for the Prisma Scanner configuration. You can add
   fields as needed to activate or deactivate behaviors. You can append the values to this file as
   shown later in this topic. Create a `values.yaml` file by using the following configuration:

    ```yaml
    ---
    namespace: DEV-NAMESPACE
    targetImagePullSecret: TARGET-REGISTRY-CREDENTIALS-SECRET
    prisma:
      url: PRISMA-URL
      tokenSecret:
        name: PRISMA-CONFIG-SECRET
    ```

   Where:

   - `DEV-NAMESPACE` is your developer namespace. To use a namespace other than the default
     namespace, ensure that the namespace exists before you install. If the namespace does not
     exist, the scanner installation fails.
   - `TARGET-REGISTRY-CREDENTIALS-SECRET` is the name of the secret that contains the credentials to
     pull an image from a private registry for scanning.
   - `PRISMA-URL` is the FQDN of your Twistlock server.
   - `PRISMA-CONFIG-SECRET` is the name of the secret you created that contains the Prisma
     configuration to connect to Prisma. This field is required.

## SCST - Store integration

The Prisma Scanner integration can work with or without the SCST - Store integration. The
`values.yaml` file is slightly different for each configuration.

When using SCST - Store integration, to persist the results found by the Prisma Scanner, you can
enable the SCST - Store integration by appending the fields to the `values.yaml` file.

The Grype, Snyk, and Prisma Scanner Integrations enable the Metadata Store. To prevent conflicts,
the configuration values are slightly different based on whether the Grype Scanner Integration is
installed or not. If Tanzu Application Platform is installed using the Full Profile, the Grype
Scanner Integration is installed unless it is explicitly excluded.

### Multiple Scanners installed

In order to find your CA secret name and authentication token secret name as needed for your
values.yaml when installing Prisma Scanner you must look at the configuration of a prior installed
scanner in the same namespace as it already exists.

For information about how the scanner was likely initially created, see
[Set up multicluster Supply Chain Security Tools (SCST) - Store](../scst-store/multicluster-setup.hbs.md)

An example `values.yaml` when there are other scanners already installed in the same `dev-namespace`
where the Prisma Scanner is installed:

```yaml
#! ...
metadataStore:
  #! The url where the Store deployment is accessible.
  #! Default value is: "https://metadata-store-app.metadata-store.svc.cluster.local:8443"
  url: "STORE-URL"
  caSecret:
    #! The name of the secret that contains the ca.crt to connect to the Store Deployment.
    #! Default value is: "app-tls-cert"
    name: "CA-SECRET-NAME"
    importFromNamespace: "" #! since both Prisma and Grype/Snyk both enable store, one must leave importFromNamespace blank
  #! authSecret is for multicluster configurations.
  authSecret:
    #! The name of the secret that contains the auth token to authenticate to the Store Deployment.
    name: "AUTH-SECRET-NAME"
    importFromNamespace: "" #! since both Prisma and Grype/Snyk both enable store, one must leave importFromNamespace blank
```

Where:

- `STORE-URL` is the URL where the Store deployment is accessible.
- `CA-SECRET-NAME` is the name of the secret that contains the ca.crt to connect to the Store
  Deployment. Default is `app-tls-cert`.
- `AUTH-SECRET-NAME` is the name of the secret that contains the authentication token to
  authenticate to the Store Deployment.

### Prisma Only Scanner Installed

For information about creating and exporting secrets for the Metadata Store CA and authentication
token referenced in the data values when installing Prisma Scanner, see
[Set up multicluster Supply Chain Security Tools (SCST) - Store](../scst-store/multicluster-setup.hbs.md).

An example `values.yaml` when no other scanner integrations installed in the same `dev-namespace`
where the Prisma Scanner is installed:

```yaml
#! ...
metadataStore:
  #! The url where the Store deployment is accessible.
  #! Default value is: "https://metadata-store-app.metadata-store.svc.cluster.local:8443"
  url: "STORE-URL"
  caSecret:
    #! The name of the secret that contains the ca.crt to connect to the Store Deployment.
    #! Default value is: "app-tls-cert"
    name: "CA-SECRET-NAME"
    #! The namespace where the secrets for the Store Deployment live.
    #! Default value is: "metadata-store"
    importFromNamespace: "STORE-SECRETS-NAMESPACE"
  #! authSecret is for multicluster configurations.
  authSecret:
    #! The name of the secret that contains the auth token to authenticate to the Store Deployment.
    name: "AUTH-SECRET-NAME"
    #! The namespace where the secrets for the Store Deployment live.
    importFromNamespace: "STORE-SECRETS-NAMESPACE"
```

Where:

- `STORE-URL` is the URL where the Store deployment is accessible.
- `CA-SECRET-NAME` is the name of the secret that contains the `ca.crt` to connect to the Store
  Deployment. Default is `app-tls-cert`.
- `STORE-SECRETS-NAMESPACE` is the namespace where the secrets for the Store Deployment live.
  Default is `metadata-store`.
- `AUTH-SECRET-NAME` is the name of the secret that contains the authentication token to
  authenticate to the Store Deployment.

### No Store Integration

If you do not want to enable the SCST - Store integration, explicitly deactivate the integration by
appending the following field to the `values.yaml` file that is enabled by default:

```yaml
# ...
metadataStore:
  url: "" # Deactivate Supply Chain Security Tools - Store integration
```

## Prepare the ScanPolicy

To prepare the ScanPolicy, use the instructions in the following sections.

### Sample ScanPolicy using Prisma Policies

The following sample ScanPolicy allows you to control whether the SupplyChain passes or fails based
on the compliance and vulnerability rules configured in the Prisma Compute Console.

The policy reads the `complianceScanPassed` and `vulnerabilityScanPassed` fields returned from
Prisma scanner output to control the results of the scan.

```yaml
apiVersion: scanning.apps.tanzu.vmware.com/v1beta1
kind: ScanPolicy
metadata:
  name: prisma-scan-policy
  labels:
    'app.kubernetes.io/part-of': 'enable-in-gui'
spec:
  regoFile: |
    package main

    import future.keywords.in

    deny[msg] {
      vulnerabilityAndComplianceScanResults := {e | e := input.bom.metadata.properties.property[_]}
      some result in vulnerabilityAndComplianceScanResults
      failedScans:= "false" in result
      failedScans
      vulnerabilityMessages := { message |
        components := {e | e := input.bom.components.component} | {e | e := input.bom.components.component[_]}
        some component in components
        vulnerabilities := {e | e := component.vulnerabilities.vulnerability} | {e | e := component.vulnerabilities.vulnerability[_]}
        some vulnerability in vulnerabilities
        ratings := {e | e := vulnerability.ratings.rating.severity} | {e | e := vulnerability.ratings.rating[_].severity}
        formattedRatings := concat(", ", ratings)
        message := sprintf("Vulnerability - Component: %s CVE: %s Severity: %s", [component.name, vulnerability.id, formattedRatings])
      }
      complianceMessages := { message |
        compliances := {e | e := input.bom.metadata.component.compliances.compliance} | {e | e := input.bom.metadata.component.compliances.compliance[_]}
        some compliance in compliances
        message := sprintf("Compliance - %s \\\nId: %s Severity: %s Category: %s", [compliance.title, compliance.id, compliance.severity, compliance.category])
      }
      combinedMessages := complianceMessages | vulnerabilityMessages
      some message in combinedMessages
      msg := message
    }
```

### Sample ScanPolicy using Local Policies

The following sample ScanPolicy allows you to control whether the SupplyChain passes or fails based
on the Prisma Scanner CycloneDX vulnerability results returned from the Prisma Scanner.

```yaml
apiVersion: scanning.apps.tanzu.vmware.com/v1beta1
kind: ScanPolicy
metadata:
  name: prisma-scan-policy
  labels:
    'app.kubernetes.io/part-of': 'enable-in-gui'
spec:
  regoFile: |
    package main

    # Accepted Values: "Critical", "High", "Medium", "Low", "Negligible", "UnknownSeverity"
    notAllowedSeverities := ["Critical", "High", "UnknownSeverity"]
    ignoreCves := []

    contains(array, elem) = true {
      array[_] = elem
    } else = false { true }

    isSafe(match) {
      severities := { e | e := match.ratings.rating.severity } | { e | e := match.ratings.rating[_].severity }
      some i
      fails := contains(notAllowedSeverities, severities[i])
      not fails
    }

    isSafe(match) {
      ignore := contains(ignoreCves, match.id)
      ignore
    }

    deny[msg] {
      comps := { e | e := input.bom.components.component } | { e | e := input.bom.components.component[_] }
      some i
      comp := comps[i]
      vulns := { e | e := comp.vulnerabilities.vulnerability } | { e | e := comp.vulnerabilities.vulnerability[_] }
      some j
      vuln := vulns[j]
      ratings := { e | e := vuln.ratings.rating.severity } | { e | e := vuln.ratings.rating[_].severity }
      not isSafe(vuln)
      msg = sprintf("CVE %s %s %s", [comp.name, vuln.id, ratings])
    }
```

Apply the YAML:

```console
kubectl apply -n $DEV-NAMESPACE -f SCAN-POLICY-YAML
```

Where:

- `DEV-NAMESPACE` is the name of the developer namespace you want to use.
- `SCAN-POLICY-YAML` is the name of your SCST - Scan YAML.

## Install Prisma Scanner

After all prerequisites are completed, install the Prisma Scanner.
See [Install another scanner for Supply Chain Security Tools - Scan](install-scanners.hbs.md).

## Self-Signed Registry Certificate

When attempting to pull an image from a registry with a self-signed certificate during image scans
additional configuration is necessary.

### Tanzu Application Platform Values Shared CA

If your `tap-values.yaml` used during install has the following shared section filled out, Prisma
Scanner uses this and enable it to connect to your registry without additional configuration.

```yaml
shared:
   ca_cert_data: | # To be passed if using custom certificates.
      -----BEGIN CERTIFICATE-----
      MIIFXzCCA0egAwIBAgIJAJYm37SFocjlMA0GCSqGSIb3DQEBDQUAMEY...
      -----END CERTIFICATE-----
```

### Secret within Developer Namespace

1. Create a secret that holds the registry's CA certificate data.

   An example of the secret:

    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: prisma-registry-cert
      namespace: dev
    type: Opaque
    data:
      ca_cert_data: BASE64_CERT
    ```

2. Update your Prisma Scanner install values.yaml. Add `caCertSecret` to the root of your
   `prisma-values.yaml` when installing Prisma Scanner. Example:

    ```yaml
    namespace: dev
    targetImagePullSecret: tap-registry
    caCertSecret: prisma-registry-cert
    ```

## Connect to Prisma through a Proxy

To connect to Prisma through a proxy, you must add `environmentVariables` configuration to your
`prisma-values.yaml`. All valid container `env` configurations are supported.

For example:

```yaml
 namespace: dev
 targetImagePullSecret: tap-registry
 environmentVariables:
 - name: HTTP_PROXY
   value: "test.proxy.com"
 - name: HTTPS_PROXY
   value: "test.proxy.com"
 - name: NO_PROXY
   value: "127.0.0.1,.svc,.svc.cluster.local,demo.app"
```

## Known Limits

OpenShift is not supported.
