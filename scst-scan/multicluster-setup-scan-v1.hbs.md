# Set up multicluster for Scan 1.0

This topic tells you how to set up your configuration to enable SCST - Scan 1.0 to connect with
SCST - Store in a multicluster deployment.

> **Important** Scan 1.0 was deprecated in Tanzu Application Platform v1.10. The default scan component
> to use in the Test and Scan supply chain is Scan 2.0. These steps are required in addition to the earlier
> steps if you are still using Scan 1.0. For more information about Scan 1.0 and Scan 2.0, see the
> [SCST - Scan component overview](../scst-scan/overview.hbs.md). For instructions to set up multicluster
> for Scan 2.0, see [Set up multicluster Artifact Metadata Repository](../scst-store/multicluster-setup.hbs.md).

## <a id='summary'></a> Procedure summary

To deploy SCST - Store in a multicluster setup:

1. [Copy the Metadata Store CA certificate from the View cluster](#copy-metadata)
1. [Copy the Metadata Store authentication token from the View cluster](#copy-auth-view)
1. [Configure the Metadata Store CA certificate and authentication token on the Build cluster](#apply-kubernetes)
1. [Configure Grype in the Build profile values file](#grype-mds-config)
1. [Export SCST - Store secrets to a developer namespace in a multicluster deployment](#export-multicluster)
1. [Install the Build and Run profiles](#install-build-run-profiles)

## <a id='copy-metadata'></a> Copy the Metadata Store CA certificate from the View cluster

With your kubectl targeted at the View cluster, copy the TLS CA certificate for Metadata Store to
the `MDS_CA_CERT` environment variable by running:

```console
MDS_CA_CERT=$(kubectl get secret -n metadata-store ingress-cert -o json | jq -r ".data.\"ca.crt\"" | base64 -d)
```

## <a id='copy-auth-view'></a> Copy the Metadata Store authentication token from the View cluster

Copy the Metadata Store authentication token into the `MDS_AUTH_TOKEN` environment variable by running:

```console
MDS_AUTH_TOKEN=$(kubectl get secrets metadata-store-read-write-client -n metadata-store -o jsonpath="{.data.token}" | base64 -d)
```

## <a id='apply-kubernetes'></a> Configure the Metadata Store CA certificate and authentication token on the Build cluster

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

This snippet contains the content of `$MDS_CA_CERT` and `$MDS_AUTH_TOKEN` copied earlier.
This content configures SCST - Scan with the Metadata Store CA certificate and authentication token.

## <a id='grype-mds-config'></a> Configure Grype in the Build profile values file

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
  [Ingress support](../scst-store/ingress.hbs.md).
- `TARGET-REGISTRY-CREDENTIALS-SECRET` is the name of the secret that contains the credentials to
  pull an image from the registry for scanning.

## <a id="export-multicluster"></a> Export SCST - Store secrets to a developer namespace in a multicluster deployment

SCST - Scan 1.0 requires SCST - Store to be configured in every developer namespace with an SCST -
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
[Cluster-specific scanner configurations](../scst-store/cluster-specific-scanner-configurations.hbs.md).

> **Important** In a multicluster configuration, copy the Metadata Store values mentioned earlier
> from the View cluster to the `values.yaml` file that you used to install the Build cluster.

## <a id='install-build-run-profiles'></a> Install the Build and Run profiles

If you came to this topic from
[Install multicluster Tanzu Application Platform profiles](../multicluster/installing-multicluster.hbs.md)
after installing the View profile, return to that topic to
[install the Build profile](../multicluster/installing-multicluster.hbs.md#install-build)
and [install the Run profile](../multicluster/installing-multicluster.hbs.md#install-run).

## <a id='resources'></a> Additional resources

- [Ingress support](../scst-store/ingress.hbs.md)
- [Custom certificate configuration](../scst-store/custom-cert.hbs.md)
