# Multicluster setup for Supply Chain Security Tools - Store

This topic describes how you can deploy Supply Chain Security Tools (SCST) - Store in a multicluster
setup, including installing multiple profiles such as, View, Build, Run, and Iterate.

## <a id='overview'></a>Overview

After installing the View profile, but before installing the Build profile and Run profile, you must
copy certain configurations from the View cluster to the Build and Run Kubernetes clusters. This
topic explains how to add these configurations which allows components in the Build and Run clusters
to communicate with SCST - Store in the View cluster.

![Screenshot of the SCST - Store in Multi Cluster Deployment](images/multicluster-deployment.jpg)

## <a id='prereqs'></a>Prerequisites

You must first install the View profile.
See [Install View profile](../multicluster/installing-multicluster.hbs.md#install-view).
This installation automatically creates the CA certificates and tokens necessary to talk to the
CloudEvent Handler.

## <a id='summary'></a>Procedure summary

To deploy Supply Chain Security Tools (SCST) - Store in a multicluster setup:

1. Copy SCST - Store CA certificates from the View cluster.
   1. Copy the Metadata Store CA certificate from the View cluster.
   2. Copy the AMR CloudEvent Handler CA certificate from the View cluster.
2. Copy SCST - Store tokens from the View cluster.
   1. Copy the Metadata Store authentication token from the View cluster.
   2. Copy the AMR CloudEvent Handler edit token from the View cluster.
3. Apply the SCST - Store CA certificates and SCST - Store tokens to the Build and Run clusters.
   1. Apply the Metadata Store CA certificate and authentication token to the Build cluster.
   2. Apply the AMR CloudEvent Handler CA certificate and edit token for the Build and Run cluster.
4. Install the Build and Run profiles.

> **Note** This topic assumes that you are using SCST - Scan 2.0, as described in
> [Add testing and scanning to your application](../getting-started/add-test-and-security.hbs.md).
> If you are still using the deprecated SCST - Scan 1.0, you must follow these steps and the
> [extra steps required for Scan 1.0](#extra-scan-1-0-steps) too.

## <a id='copy-ca-cert'></a>Copy SCST - Store CA certificates from the View cluster

To copy SCST - Store CA certificates from the View cluster, you must copy the Metadata Store CA
certificate and the AMR CloudEvent Handler CA certificate from the View cluster.

### <a id='copy-metadata'></a>Copy Metadata Store CA certificate from the View cluster

With your kubectl targeted at the View cluster, you can get Metadata Store's TLS CA certificate.

```console
MDS_CA_CERT=$(kubectl get secret -n metadata-store ingress-cert -o json | jq -r ".data.\"ca.crt\"" | base64 -d)
```

### <a id='copy-ceh-ca'></a>Copy AMR CloudEvent Handler CA certificate data from the View cluster

With your kubectl targeted at the View cluster, you can get AMR CloudEvent Handler's TLS CA
certificate's data.

```console
CEH_CA_CERT_DATA=$(kubectl get secret -n metadata-store amr-cloudevent-handler-ingress-cert -o json | jq -r ".data.\"ca.crt\"" | base64 -d)
```

## <a id='copy-token'></a>Copy SCST - Store authentication tokens from the View cluster

To copy SCST - Store tokens from the View cluster, you must copy the Metadata Store authentication
token and the AMR CloudEvent Handler edit token from the View cluster.

### <a id='copy-auth-view'></a>Copy Metadata Store authentication token from the View cluster

Copy the Metadata Store authentication token into an environment variable:

```console
MDS_AUTH_TOKEN=$(kubectl get secrets metadata-store-read-write-client -n metadata-store -o jsonpath="{.data.token}" | base64 -d)
```

You use this environment variable in the next step.

### <a id='copy-ceh-token'></a>Copy AMR CloudEvent Handler edit token from the View cluster

Copy the AMR CloudEvent Handler token into an environment variable:

```console
CEH_EDIT_TOKEN=$(kubectl get secrets amr-cloudevent-handler-edit-token -n metadata-store -o jsonpath="{.data.token}" | base64 -d)
```

You use this environment variable in the next step.

## <a id='apply-build'></a>Apply the SCST - Store CA certificates and SCST - Store tokens to the Build and Run clusters

After you copy the certificate and tokens, apply them to the Build and Run clusters before deploying
the profiles.

Build cluster:

- Metadata Store CA certificate
- Metadata Store authentication token
- CloudEvent Handler CA certificate
- CloudEvent Handler edit token

Run cluster:

- CloudEvent Handler CA certificate
- CloudEvent Handler edit token

### <a id='apply-ceh-ca-token'></a>Apply the CloudEvent Handler CA certificate data and edit token to the Build and Run clusters

You can apply the CloudEvent Handler CA certificate and edit the token to the Build and Run clusters.
These values must be accessible during the Build and Run profile deployments.

1. Update your kubectl to target the Build cluster.
1. If you already installed Build Cluster you can skip this step. Create a namespace for the
   CloudEvent Handler CA certificate and edit token.

   ```console
   kubectl create ns amr-observer-system
   ```

1. Update the Build profile `values.yaml` file to add the following snippet. It configures the CA
   certificate and endpoint. In `amr.observer.cloudevent_handler.endpoint` you specify the location
   of the CloudEvent Handler which was deployed to the View cluster. In `amr.observer.ca_cert_data`
   you paste the contents of `$CEH_CA_CERT_DATA` which you copied earlier.

   ```console
   amr:
     observer:
       auth:
         kubernetes_service_accounts:
           enable: true
       cloudevent_handler:
         endpoint: https://amr-cloudevent-handler.<VIEW-CLUSTER-INGRESS-DOMAIN>
       ca_cert_data: |
           <CONTENTS OF $CEH_CA_CERT_DATA>
   ```

1. Create a secret to store the CloudEvent Handler edit token. This uses the `CEH_EDIT_TOKEN`
   environment variable.

   ```console
   kubectl create secret generic amr-observer-edit-token \
     --from-literal=token=$CEH_EDIT_TOKEN -n amr-observer-system
   ```

1. Repeat the earlier steps, but configure kubectl to target the Run cluster instead of the Build
   cluster.

After all the steps are done, both the Build and Run clusters each have a CloudEvent Handler CA
certificate and edit token named `amr-observer-edit-token` in the namespaces
`metadata-store-secrets` and `amr-observer-system`. Now you are ready to deploy the Build and Run
profiles.

## <a id='install-build-run-profiles'></a>Install the Build and Run profiles

If you came to this topic from
[Install multicluster Tanzu Application Platform profiles](../multicluster/installing-multicluster.hbs.md)
after installing the View profile, return to that topic to
[install the Build profile](../multicluster/installing-multicluster.hbs.md#install-build)
and [install the Run profile](../multicluster/installing-multicluster.hbs.md#install-run).

## <a id='extra-scan-1-0-steps'></a>Extra steps required for Scan 1.0

Scan 1.0 was deprecated in Tanzu Application Platform v1.10. The default scan component to use in
the Test and Scan supply chain is Scan 2.0. These steps are required in addition to the earlier steps
if you are still using Scan 1.0. For more information about Scan 1.0 and Scan 2.0, see the
[SCST - Scan component overview](../scst-scan/overview.hbs.md)

### <a id='apply-kubernetes'></a> Configure SCST - Scan with the Metadata Store CA certificate and authentication token on the Build cluster

Within the Build profile `values.yaml` file, add the following snippet:

```yaml
scanning:
  metadataStore:
    exports:
      ca:
        pem: |
          <CONTENTS OF $MDS_CA_CERT>
      auth:
        token: <CONTENTS OF $MDS_AUTH_TOKEN>
```

This snippet contains the content of `$MDS_CA_CERT` and `$MDS_AUTH_TOKEN` copied in an earlier step.
This content configures SCST - Scan with the Metadata Store CA certificate and authentication token.

### <a id='grype-mds-config'></a>How to configure Grype in the Build profile values file

The Build profile `values.yaml` file uses the secrets you created to configure the Grype scanner
that communicates with SCST - Store. After performing a vulnerabilities scan, the Grype scanner
sends the scan result to SCST - Store.

For example:

```yaml
...
grype:
  targetImagePullSecret: "TARGET-REGISTRY-CREDENTIALS-SECRET"
  metadataStore:
    url: METADATA-STORE-URL-ON-VIEW-CLUSTER # URL with http / https
    caSecret:
        name: store-ca-cert
        importFromNamespace: metadata-store-secrets # Must match with ingress-cert.data."ca.crt" of store on view cluster
    authSecret:
        name: store-auth-token # Must match with valid store token of metadata-store on view cluster
        importFromNamespace: metadata-store-secrets
...
```

Where:

- `METADATA-STORE-URL-ON-VIEW-CLUSTER` is the ingress URL of SCST - Store deployed to the View
  cluster. For example, `https://metadata-store.example.com`. For more information, see
  [Ingress support](ingress.hbs.md).
- `TARGET-REGISTRY-CREDENTIALS-SECRET` is the name of the secret that contains the credentials to
  pull an image from the registry for scanning.

### <a id="export-multicluster"></a> Exporting SCST - Store secrets to a developer namespace in a Tanzu Application Platform multicluster deployment

SCST - Scan 1.0 required SCST - Store to be configured in every developer namespace with an SCST -
Store certificate and authentication token.

To export secrets by creating `SecretExport` resources on the developer namespace:

1. Verify that you created and populated the `metadata-store-secrets` namespace.
1. Create the `SecretExport` resources by running:

   ```console
   cat <<EOF | kubectl apply -f -
   ---
   apiVersion: secretgen.carvel.dev/v1alpha1
   kind: SecretExport
   metadata:
     name: store-ca-cert
     namespace: metadata-store-secrets
   spec:
     toNamespaces: [DEV-NAMESPACES]
   ---
   apiVersion: secretgen.carvel.dev/v1alpha1
   kind: SecretExport
   metadata:
     name: store-auth-token
     namespace: metadata-store-secrets
   spec:
     toNamespaces: [DEV-NAMESPACES]
   EOF
   ```

   Where `DEV-NAMESPACES` is an array of developer namespaces where the Metadata Store secrets are
   exported.

For information about `metadata` configuration, see
[Cluster-specific scanner configurations](cluster-specific-scanner-configurations.hbs.md).

> **Important** Although in a single cluster configuration using of Namespace Provisioner
> is recommeneded, however in any multicluster configuration properties mentioned above should
> be taken from the view cluster manually and copied to the `values.yaml` by which the build
> cluster would be installed.

## <a id='resources'></a> Additional resources

- [Ingress support](ingress.hbs.md)
- [Custom certificate configuration](custom-cert.hbs.md)
