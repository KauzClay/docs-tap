# Security details for Supply Chain Security Tools - Store

This topic tells you about the security details for Supply Chain Security Tools (SCST) - Store.

## <a id='app-sec'></a>Application security

The following sections describe application security in detail.

### <a id='tls-encrypt'></a> TLS encryption

SCST - Store requires a TLS connection. If certificates are not provided, the application does not
start. SCST - Store supports TLS v1.2 and TLS v1.3. It does not support TLS v1.0 to prevent a
downgrade attack. TLS v1.0 is prohibited under the Payment Card Industry Data Security Standard (PCI
DSS).

#### <a id='crypto-al'></a> Cryptographic algorithms

Elliptic Curve:

```text
CurveP521
CurveP384
CurveP256
```

Cipher Suites:

```text
TLS_AES_128_GCM_SHA256
TLS_AES_256_GCM_SHA384
TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
```

### <a id='acc-control'></a> Access controls

SCST - Store uses [kube-rbac-proxy](https://github.com/brancz/kube-rbac-proxy) as the only entry
point to its API. Authentication and authorization must be completed by using `kube-rbac-proxy`
before its API is accessible.

#### <a id='auth-token'></a> Authentication

`kube-rbac-proxy` uses
[Token Review](https://kubernetes.io/docs/reference/access-authn-authz/authentication/) to verify
that the token is valid. `Token Review` is a Kubernetes API to ensure that a trusted vendor issued
the access token that the user provided. To issue an access token by using Kubernetes, create a
Kubernetes service account and then retrieve the corresponding generated secret for the access
token.

To create a service account and use its access token, see
[Retrieve and create service accounts for SCST - Store](create-service-account.hbs.md).

#### <a id='auth-api'></a> Authorization

The `kube-rbac-proxy` uses
[Subject Access Review](https://kubernetes.io/docs/reference/access-authn-authz/authorization/) to
ensure that you can access specific operations. Subject Access Review is a Kubernetes API that uses
[Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) to verify that
you can perform specific actions. For more information, see
[Retrieve and create service accounts for SCST - Store](create-service-account.hbs.md).

There are two supported cluster roles:

- `Read Only`
- `Read and Write`

These cluster roles are deployed by default. Additionally, a service account is created and bound to
the `Read and Write` cluster role by default. If you do not want this service account, set the
`add_default_rw_service_account` property to `false` in the `metadata-store-values.yaml` file during
deployment. See [Install SCST - Store](install-scst-store.hbs.md).

There is no default service account bound to the `Read Only` cluster role. You must create your
service account and cluster role binding to bind to the `Read Only` role.

> **Important** There is no support for roles with access to only specific types of resources. For
> example, images, packages, and vulnerabilities.

## <a id='contain-sec'></a> Container security

This section describes container security.

### <a id='non-root'></a> Non-root user

All shipped containers do not use root user accounts or accounts with root access. Use Kubernetes
Security Context to ensure that applications do not run with root users.

Security Context for the API server:

```text
allowPrivilegeEscalation: false
runAsUser: 65532
fsGroup: 65532
```

Security Context for the PostgreSQL database pod:

```text
allowPrivilegeEscalation: false
runAsUser: 999
fsGroup: 999
```

> **Note** `65532` is the UUID for the nobody user. `999` is the UUID for the PostgreSQL user.

## <a id='sec-scan'></a> Security scanning

There are two types of security scans that are performed before every release.

### <a id='sast'></a> Static Application Security Testing (SAST)

A Coverity Scan is run on the source code of the API server, the CLI, and all their dependencies.
There are no outstanding high or critical items at the time of release.

### <a id='sca'></a> Software Composition Analysis (SCA)

A Black Duck scan is run on the compiled binary to detect vulnerabilities and license data. There
are no outstanding high or critical items at the time of release.

A Grype scan is run against the source code and the compiled container to detect dependency
vulnerabilities. There are no outstanding high or critical items at the time of release.
