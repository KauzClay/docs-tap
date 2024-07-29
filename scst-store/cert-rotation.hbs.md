# Certificate rotation for Supply Chain Security Tools - Store

This topic tells how you to rotate TLS certificates for Supply Chain Security Tools (SCST) - Store.

## Certificates

By default, the `use_cert_manager` setting is set to `"true"`. When the setting `use_cert_manager`
is `"true"` the Store uses `cert-manager` to generate a certificate authority (CA) certificate, an
API certificate, and a database Certificate.

To see these certificates run:

```console
kubectl get certificate -n metadata-store
```

For example:

```console
$ kubectl get certificate -n metadata-store
NAME                    READY   SECRET                  AGE
app-tls-ca-cert         True    app-tls-ca-cert         38d
app-tls-cert            True    app-tls-cert            38d
postgres-db-tls-cert    True    postgres-db-tls-cert    38d
```

`cert-manager` automatically rotates rhe earlier certificates. The Store can run these certificates
automatically after `cert-manager` rotates them.

If the environment is a [multi-cluster](multicluster-setup.hbs.md) setup, the operator must manually
copy the new CA certificate to the build cluster.

## Certificate duration setting

In `tap-values.yaml`, `api_cert_duration`, `api_cert_renew_before`, `ca_cert_duration`, and
`ca_cert_renew_before` are configurable as shown in the following YAML:

```yaml
metadata_store:
  ca_cert_duration: CA-DURATION
  ca_cert_renew_before: CA-RENEW
  api_cert_duration: API-DURATION
  api_cert_renew_before: API-RENEW
```

Where:

- `CA-DURATION` is the duration that the CA certificate is valid for. It must be entered as `h`,
  `m`, or `s`. The default value is `8760h`.
- `CA-RENEW` is time period before the expiry of the CA certificate is renewed. It must be entered
  as `h`, `m`, or `s`. The default value is `1h`.
- `API-DURATION` is the duration that the API certificate is valid for. It must be entered as `h`,
  `m`, or `s`. The default value is `2160h`.
- `API-RENEW` is the time period before the expiry time of the API certificate is renewed. It must
  be entered as `h`, `m`, or `s`. The default value is `24h`.

> **Important** The `*_cert_duration` and the corresponding `*_renew_before` settings must not be
> close together. For more information, see the
> [cert-manager documentation](https://cert-manager.io/docs/usage/certificate/#renewal). This can
> lead to a renewal loop. The `*_cert_duration` must be greater than the corresponding
> `*_renew_before`. The earlier settings only take effect when `use_cert_manager` is `"true"`. If
> the `use_cert_manager` is not set, it is `"true"` by default.z