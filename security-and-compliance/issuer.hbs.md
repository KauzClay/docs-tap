# Secure exposed ingress endpoints in Tanzu Application Platform

This topic tells you how to secure exposed ingress endpoints with TLS in Tanzu Application Platform
(commonly known as TAP).

Tanzu Application Platform exposes ingress endpoints so that:

- Platform operators and application developers can interact with the platform
- End users can interact with applications running on the platform

There are two ways of configuring ingress certificates to secure endpoints with TLS, such as
`https://`:

- [Shared ingress issuer](#shared)
- [Component-level configuration](#component)

## <a id="shared"></a> Shared ingress issuer

Ingress communication uses TLS by default. VMware recommends using a shared ingress issuer for
issuing ingress certificates on Tanzu Application Platform.

The shared ingress issuer is an on-platform representation of a certificate authority. It provides a
method to set up TLS for the entire platform. The shared ingress issuer issues ingress certificates
for all participating components.

### <a id="default"></a> Default self-signed ingress issuer

By default, Tanzu Application Platform installs and uses a self-signed CA ingress issuer for all
components. The default ingress issuer is a self-signed `cert-manager.io/v1/ClusterIssuer` provided
by the [cert-manager package](../cert-manager/about.hbs.md).

The ingress issuer is designated by the Tanzu Application Platform configuration value
`shared.ingress_issuer`. It is `tap-ingress-selfsigned` by default.

> **Caution** The default ingress issuer is appropriate for testing and evaluation. VMware
> recommends that you replace the default self-signed issuer with your own issuer.

### Default self-signed issuer limitations

The default ingress issuer represents a self-signed certificate authority. This is not problematic
as far as security is concerned, however, it is not included in any configured trust chain. As a
result, nothing trusts the default ingress issuer implicitly, not even Tanzu Application Platform
components. Although the issued certificates are valid in principle, they are rejected. Even your
browser, for example, rejects them. Furthermore, some interactions between components are not
functional by default.

### <a id="prerequisites"></a> Default self-signed issuer prerequisites

To use the Tanzu Application Platform ingress issuer, your certificate authority must be
representable by a cert-manager `ClusterIssuer`. You need one of the following:

- Your own CA certificate
- An ACME, Venafi, or Vault-based issuer, such as LetsEncrypt, as your CA
- An [external](https://cert-manager.io/docs/configuration/external/) cert-manager `ClusterIssuer`
  representing your CA

Without one of these, you cannot use the ingress issuer, but you can still configure TLS for
components. For more information, see [Ingress certificates inventory](inventory.hbs.md).

> **Important** If `cert-manager.tanzu.vmware.com` is excluded from the installation then
> `tap-ingress-selfsigned` is not installed either. In this case, bring your own ingress issuer.

### <a id="trust-self-signed"></a> Trust the default self-signed issuer

You can trust the default ingress issuer by including `tap-ingress-selfsigned`'s certificate in the
Tanzu Application Platform trusted CA certificates and your device's certificate chain.

> **Caution** VMware discourages this approach. VMware recommends replacing the default ingress
> issuer instead.

To trust the default ingress issuer:

1. Obtain `tap-ingress-selfsigned`'s PEM-encoded certificate by running:

   ```console
   kubectl get secret \
     tap-ingress-selfsigned-root-ca \
     --namespace cert-manager \
     --output go-template='\{{ index .data "tls.crt" | base64decode }}'
   ```

1. Add the certificate to [custom CA certificates](custom-ca-certificates.hbs.md) by appending it to
   `shared.ca_cert_data` and applying Tanzu Application Platform's installation values.

1. Add the certificate to your device's trust chain. The trust chain varies depending on your
   operating system and privileges.

### <a id="replace"></a> Replace the default ingress issuer

You can replace Tanzu Application Platform's default ingress issuer with any other
[cert-manager-compliant ClusterIssuer](https://cert-manager.io/docs/configuration/).

To replace the default ingress issuer:

Custom CA
: Complete the following steps:

  1. Obtain CA certificates and a private key.

  1. Create a `Secret` and `ClusterIssuer` that represent your CA on the platform:

     ```yaml
     ---
     apiVersion: v1
     kind: Secret
     type: kubernetes.io/tls
     metadata:
       name: my-company-ca
       namespace: cert-manager
     stringData:
       tls.crt: #! your CA's PEM-encoded certificate
       tls.key: #! your CA's PEM-encoded private key

     ---
     apiVersion: cert-manager.io/v1
     kind: ClusterIssuer
     metadata:
       name: my-company
     spec:
       ca:
         secretName: my-company-ca
     ```

  1. Set `shared.ingress_issuer` to the name of your issuer:

     ```yaml
     #! my-tap-values.yaml
     #! ...
     shared:
       ingress_issuer: my-company-ca
     #! ...
     ```

  1. Apply the Tanzu Application Platform installation values file. When the configuration is
     applied, components obtain certificates from the new issuer and serve them.

LetsEncrypt production
: Complete the following steps:

  1. Public CAs, such as LetsEncrypt, record signed certificates in publicly-available certificate
     logs for [certificate transparency](https://certificate.transparency.dev/). Verify that you are
     okay with this before using LetsEncrypt.

  1. Verify that the LetsEncrypt production API
     [rate limits](https://letsencrypt.org/docs/rate-limits/) suit your needs.

  1. Ensure that your `shared.ingress_domain` is accessible from the Internet.

  1. Adjust
     [.spec.acme.solvers](https://cert-manager.io/docs/configuration/acme/#solving-challenges) if
     your setup requires that.

  1. Replace `.spec.acme.email` with the email address that will receive notices for certificates
     from LetsEncrypt.

     > **Caution** ACME HTTP01 challenges can fail under some conditions. For more information,
     > see [ACME challenges](../cert-manager/acme-challenges.hbs.md).

  1. Create a `ClusterIssuer` for the [LetsEncrypt](https://letsencrypt.org) production API:

     ```yaml
     ---
     apiVersion: cert-manager.io/v1
     kind: ClusterIssuer
     metadata:
       name: letsencrypt-production
     spec:
       acme:
         email: certificate-notices@my-company.com
         privateKeySecretRef:
           name: letsencrypt-production
         server: https://acme-v02.api.letsencrypt.org/directory
         solvers:
           - http01:
               ingress:
                 class: contour
     ```

  1. Set `shared.ingress_issuer` to the name of your issuer:

     ```yaml
     #! my-tap-values.yaml
     #! ...
     shared:
       ingress_issuer: letsencrypt-production
     #! ...
     ```

  1. Apply Tanzu Application Platform installation values. When the configuration is applied,
     components obtain certificates from the new issuer and serve them.

LetsEncrypt staging
: Complete the following steps:

  1. Public CAs, such as LetsEncrypt, record signed certificates in publicly-available certificate
     logs for [certificate transparency](https://certificate.transparency.dev/). Verify that you are
     okay with this before using LetsEncrypt.

  1. LetsEncrypt's staging API is not a publicly-trusted CA. Add its certificate to your device's
     trust chain and
     [Tanzu Application Platform's custom CA certificates](custom-ca-certificates.hbs.md).

  1. Ensure that your `shared.ingress_domain` is accessible from the Internet.

  1. Adjust
     [.spec.acme.solvers](https://cert-manager.io/docs/configuration/acme/#solving-challenges) if
     your setup requires that.

  1. Replace `.spec.acme.email` with the email address that will receive notices for certificates
     from LetsEncrypt.

     > **Caution** ACME HTTP01 challenges can fail under some conditions. For more information, see
     > [ACME challenges](../cert-manager/acme-challenges.hbs.md).

  1. Create a `ClusterIssuer` for the LetsEncrypt staging API:

     ```yaml
     ---
     apiVersion: cert-manager.io/v1
     kind: ClusterIssuer
     metadata:
       name: letsencrypt-staging
     spec:
       acme:
         email: certificate-notices@my-company.com
         privateKeySecretRef:
           name: letsencrypt-production
         server: https://acme-staging-v02.api.letsencrypt.org/directory
         solvers:
           - http01:
               ingress:
                 class: contour
     ```

  1. Set `shared.ingress_issuer` to the name of your issuer:

     ```yaml
     #! my-tap-values.yaml
     #! ...
     shared:
       ingress_issuer: letsencrypt-staging
     #! ...
     ```

  1. Apply Tanzu Application Platform installation values. After the configuration is applied,
     components obtain certificates from the new issuer and serve them.

Other
: Complete the following steps:

  You can use any other cert-manager-supported `ClusterIssuer` as an ingress issuer for Tanzu
  Application Platform.

  cert-manager supports a host of in-tree and out-of-tree issuers. For more information, see
  cert-manager's [documentation of issuers](https://cert-manager.io/docs/configuration/).

  1. Set `shared.ingress_issuer` to the name of your issuer:

     ```yaml
     #! my-tap-values.yaml
     #! ...
     shared:
       ingress_issuer: my-company-ca
     #! ...
     ```

  1. Apply Tanzu Application Platform's installation values. After the configuration is applied,
     components obtain certificates from the new issuer and serve them.

There are many ways and tools to assert that new certificates are issued and served. It is best to
connect to one of the ingress endpoints and inspect the certificate it serves.

The `openssl` command-line utility is available on most operating systems. To retrieve the
certificate from an ingress endpoint and show its text representation, run:

```console
# replace tap.example.com with your Tanzu Application Platform installation's ingress domain
openssl s_client -showcerts -servername tap-gui.tap.example.com -connect \
tap-gui.tap.example.com:443 <<< Q | openssl x509 -text -noout
```

Alternatively, use a browser to navigate to the ingress endpoint and click the lock icon in the
address bar to inspect the certificate.

## <a id="component"></a> Component-level configuration

In some situations, depending on [prerequisites](#prerequisites), the shared ingress issuer is not
the right choice. You can override configuration of TLS and certificates per component. A
component's ingress and TLS configuration take precedence over the shared ingress issuer.

For a list of components with ingress and how to customize them, see
[Plan Ingress certificates inventory in Tanzu Application Platform](inventory.hbs.md).

Tanzu Application Platform has limited support for wildcard certificates. For more information,
see [Wildcards](inventory.hbs.md#wildcards).

### <a id="deactivate"></a> Deactivate TLS for ingress

Although VMware discourages it, you can deactivate the ingress issuer by setting
`shared.ingress_issuer: ""`.