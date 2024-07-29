# Custom certificate configuration for Supply Chain Security Tools - Store

This topic describes how you can configure the following certificates for Supply Chain Security
Tools (SCST) - Store.

## Default configuration

By default, SCST - Store creates a self-signed certificate and TLS communication is automatically
enabled.

If [ingress support](ingress.hbs.md) is enabled, SCST - Store installation creates an HTTPProxy
entry with host-routing by using the qualified name `metadata-store.<ingress_domain>`. For example,
`metadata-store.example.com`. The created route supports HTTPS communication using the self-signed
certificate with the same subject `Alternative Name`.

## (Optional) Set up a custom ingress TLS certificate

To configure TLS to use a custom certificate:

1. Place the certificates in the secret.
1. Edit the `tap-values.yaml` to use this secret.

### Place the certificates in a secret

To place the certificates in a secret:

1. Create the certificate secret before deploying SCST - Store.
1. Create a Kubernetes object with the kind `Secret` and the type `kubernetes.io/tls`.

### Edit the `tap-values.yaml` to use the secret

In `tap-values.yaml`, configure the metadata store to use the `namespace` and `secretName` from the
secret you just created, as in this example:

```yaml
metadata_store:
  tls:
    namespace: "NAMESPACE"
    secretName: "SECRET-NAME"
```

Where:

- `NAMESPACE` is the targeted namespace for secret consumption by the HTTPProxy.
- `SECRET-NAME` is the name of secret for consumption by the HTTPProxy.

## Additional resources

- [Ingress support](ingress.hbs.md)
- [TLS configuration](tls-configuration.hbs.md)