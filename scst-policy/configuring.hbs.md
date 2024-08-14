# Configure Supply Chain Security Tools - Policy

This topic describes how you can configure Supply Chain Security Tools (SCST) - Policy. SCST -
Policy requires extra configuration steps to verify your container images.

## <a id="admission-of-images"></a> Admit images

An image is admitted after it is validated against a policy with a matching image pattern and where
at least one valid signature is obtained from the authorities provided in a matched
`ClusterImagePolicy`. For more information, see
[Create a ClusterImagePolicy resource](#create-cip-resource) later in the topic.

If more than one policy exists with a matching image pattern, all of the policies must have at least
one passing authority for the image.

## <a id="including-namespaces"></a> Include namespaces

The Policy Controller only validates resources in include namespaces. Include a namespace resource
by adding the label `policy.sigstore.dev/include: "true"`. For example, by running:

```console
$ kubectl label namespace my-secure-namespace policy.sigstore.dev/include=true
```

> **Caution** Without a Policy Controller `ClusterImagePolicy` applied, there are fallback behaviors
> where images are validated against the public Sigstore Rekor and Fulcio servers by using a keyless
> authority flow.
>
> Therefore, if the deploying image is signed publicly by a third party using the
> keyless authority flow, the image is admitted because it can validate against the public Rekor and
> Fulcio. To avoid this behavior, develop and apply a `ClusterImagePolicy` that applies to the
> images being deployed in the namespace.

## <a id="create-cip-resource"></a> Create a `ClusterImagePolicy` resource

The cluster image policy is a custom resource containing properties described in the following
sections.

### <a id="cip-images"></a> Include `images`

In a `ClusterImagePolicy`, `spec.images` specifies a list of glob matching patterns. These patterns
are matched against the image digest in `PodSpec` for resources attempting deployment.

Policy Controller defines the following globs by default:

- If `*` is specified, the `glob` matching behavior is `index.docker.io/library/*`.
- If `*/*` is specified, the `glob` matching behavior is `index.docker.io/*/*`.

With these default values, you require the `glob` pattern `**` to match against all images.
If your image is hosted on Docker Hub, include `index.docker.io` as the host for the glob.

This is a sample `ClusterImagePolicy` that matches against all images using glob:

```yaml
apiVersion: policy.sigstore.dev/v1beta1
kind: ClusterImagePolicy
metadata:
  name: image-policy
spec:
  images:
  - glob: "**"
```

### <a id="cip-mode"></a> Configure `mode`

In a `ClusterImagePolicy`, `spec.mode` specifies the action of a policy:

- `enforce`: This is the default behavior. If the policy fails to validate the image, the policy
  fails.
- `warn`: If the policy fails to validate the image, validation error messages are converted to
  warnings and the policy passes.

This is a sample of a `ClusterImagePolicy` that has `warn` mode configured:

```yaml
---
apiVersion: policy.sigstore.dev/v1beta1
kind: ClusterImagePolicy
metadata:
  name: POLICY-NAME
spec:
  mode: warn
```

Where `POLICY-NAME` is the name of the policy you want to configure your `ClusterImagePolicy` with.

When `enforce` mode is set, an image that fails validation is not admitted.

Sample output message:

```console
error: failed to patch: admission webhook "policy.sigstore.dev" denied the request: validation failed: failed policy: POLICY-NAME: spec.template.spec.containers[0].image
IMAGE-REFERENCE signature key validation failed for authority authority-0 for IMAGE-REFERENCE: GET IMAGE-SIGNATURE-REFERENCE: DENIED: denied; denied
failed policy: POLICY-NAME: spec.template.spec.containers[1].image
IMAGE-REFERENCE signature key validation failed for authority authority-0 for IMAGE-REFERENCE: GET IMAGE-SIGNATURE-REFERENCE: DENIED: denied; denied
```

When `warn` mode is set, an image that fails validation is admitted.

Sample output message:

```console
Warning: failed policy: POLICY-NAME: spec.template.spec.containers[0].image
Warning: IMAGE-REFERENCE signature key validation failed for authority authority-0 for IMAGE-REFERENCE: GET IMAGE-SIGNATURE-REFERENCE: DENIED: denied; denied
Warning: failed policy: POLICY-NAME: spec.template.spec.containers[1].image
Warning: IMAGE-REFERENCE signature key validation failed for authority authority-0 for IMAGE-REFERENCE: GET IMAGE-SIGNATURE-REFERENCE: DENIED: denied; denied
```

If you do not want a `Warning` output message, you can configure a `static.action` `pass` authority
to allow expected unsigned images. For example, you might want to allow unsigned images if your
policy controller runs on a development environment and you need to iterate quickly. For information
about static action authorities, see [Static Action](#cip-static-action).

### <a id="cip-match"></a> Configure `match`

You can use `match` to filter resources by group, version, kind, or labels in a selected
namespace to enforce the defined policy. If the list of matching resources is empty, all core
resources are used by default.

For example, you can filter all `v1` `cronjobs` with the label `app: tap` in a namespace that is
labeled for policy enforcement:

```yaml
spec:
  match:
  - group: batch
    resource: cronjobs
    version: v1
    selector:
      matchLabels:
        app: tap
```

### <a id="cip-authorities"></a> Configure `authorities`

Authorities listed in the `authorities` block of the `ClusterImagePolicy` are `key` or `keyless`
specifications.

### <a id="key-authority"></a> Configure `key`

Each `key` authority can contain a PEM-encoded ECDSA public key, a `secretRef`, or a `kms` path. For
example:

```yaml
spec:
  authorities:
    - key:
        data: |
          -----BEGIN PUBLIC KEY-----
          ...
          -----END PUBLIC KEY-----
    - key:
        secretRef:
          name: secretName
    - key:
        kms: KMSPATH
```

Where `KMSPATH` is the name of the KMS path you want to configure in your key authority.

> **Important** Only ECDSA public keys are supported.

The policy resyncs with KMS referenced every 10 hours. Any updates to the secret in KMS is pulled in
during the refresh. To force a resync, the policy must be deleted and re-created.

The secret referenced in `key.secretRef.name` must be created in the `cosign-system` namespace or
the namespace where the Policy Controller is installed. This secret must only contain one `data`
entry with the public key.

### <a id="keyless-authority"></a> Configure `keyless`

Each keyless authority can contain a Fulcio URL, a Rekor URL, a certificate, or an array of
identities.

> **Note** Keyless support is deactivated by default. For more information, see
> [Install Supply Chain Security Tools - Policy Controller](install-scst-policy.hbs.md).

Identities are represented with a combination of `issuer` or `issuerRegExp` with `subject` or
`subjectRegExp`.

- `issuer`: Defines the issuer for this identity.
- `issuerRegExp`: Specifies a regular expression to match the issuer for this identity.
- `subject`: Defines the subject for this identity.
- `subjectRegExp`: Specifies a regular expression to match the subject for this identity.

An example of keyless authority structure:

```yaml
spec:
  authorities:
    - keyless:
        url: https://fulcio.example.com
        ca-cert:
          data: Certificate Data
        identities:
          - issuer: https://accounts.google.com
            subjectRegExp: .*@example.com
          - issuer: https://token.actions.githubusercontent.com
            subject: https://github.com/mycompany/*/.github/workflows/*@*
      ctlog:
        url: https://rekor.example.com
    - keyless:
        url: https://fulcio.example.com
        ca-cert:
          secretRef:
            name: secretName
        identities:
          - issuerRegExp: .*kubernetes.default.*
            subjectRegExp: .*kubernetes.io/namespaces/default/serviceaccounts/default
```

The authorities are evaluated by using the `any of` operator to admit container images. For each
pod, the Policy Controller iterates over the list of containers and init containers. For every
policy that matches against the images, they must each have at least one valid signature obtained by
using the authorities specified. If an image does not match any policy, the Policy Controller does
not admit the image.

### <a id="cip-static-action"></a> Configure `static.action`

`ClusterImagePolicy` authorities are configured to always `pass` or `fail` with `static.action`.

This is a sample `ClusterImagePolicy` with static action `fail`:

```yaml
apiVersion: policy.sigstore.dev/v1beta1
kind: ClusterImagePolicy
metadata:
  name: POLICY-NAME
spec:
  authorities:
  - static:
      action: fail
```

Where `POLICY-NAME` is the name of the policy you want to configure your `ClusterImagePolicy` with.

This is sample output of static action `fail`:

```console
error: failed to patch: admission webhook "policy.sigstore.dev" denied the request: validation failed: failed policy: POLICY-NAME: spec.template.spec.containers[0].image
IMAGE-REFERENCE disallowed by static policy
failed policy: POLICY-NAME: spec.template.spec.containers[1].image
IMAGE-REFERENCE disallowed by static policy
```

Images that are unsigned in a namespace with validation enabled are admitted with an authority with
the static action `pass`.

This applies when you are configuring a policy with `static.action` `pass` for `tap-packages`
images. Another policy is then configured to validate signed images produced by Tanzu Build Service.
This allows images from `tap-packages`, which are unsigned and required by the platform, to be
admitted while still validating signed built images from Tanzu Build Service. For more information,
see
[Configure your supply chain to sign and verify your image builds](../getting-started/config-supply-chain.hbs.md#config-sc-to-img-builds).

If you want `Warning` messages for admitted images where validation failed, you can configure a
policy with `warn` mode and valid authorities. For information about `ClusterImagePolicy` modes, see
[Mode](#cip-mode) earlier in this topic.

## <a id='provide-creds-for-package'></a> Provide credentials for the package

There are three ways the package reads credentials to authenticate to registries protected by
authentication:

- Reading `imagePullSecrets` directly from the resource being admitted. For more information, see
  [Container image pull secrets](https://kubernetes.io/docs/concepts/configuration/secret/#using-imagepullsecrets)
  in the Kubernetes documentation.

- Reading `imagePullSecrets` from the service account the resource is running as. For more
  information, see
  [Arranging for imagePullSecrets to be automatically attached](https://kubernetes.io/docs/concepts/configuration/secret/#arranging-for-imagepullsecrets-to-be-automatically-attached)
  in the Kubernetes documentation.

- Reading a `secretRef` from the `ClusterImagePolicy` resource's `signaturePullSecrets` when
  specifying the cosign signature source.

Authentication can fail for the following scenarios:

- An invalid credential is specified in the `imagePullSecrets` of the resource or in the service
  account the resource runs as.
- An invalid credential is specified in the `ClusterImagePolicy` `signaturePullSecrets` field.

### <a id="provide-pol-auth-secrets"></a> Provide secrets for authentication in your policy

You can provide secrets for authentication as part of the policy configuration. The `oci` location
is the image location or a remote location where signatures are configured to be stored during
signing. `signaturePullSecrets` is available in the `cosign-system` namespace or the namespace
where the Policy Controller is installed.

By default, `imagePullSecrets` from the resource or service account is used while the default `oci`
location is the image location.

See the following example:

```yaml
spec:
  authorities:
    - key:
        data: |
          -----BEGIN PUBLIC KEY-----
          ...
          -----END PUBLIC KEY-----
      source:
        - oci: registry.example.com/project/signature-location
          signaturePullSecrets:
            - name: MY-SECRET
    - keyless:
        url: https://fulcio.example.com
      source:
        - oci: registry.example.com/project/signature-location
          signaturePullSecrets:
            - name: MY-SECRET
```

Where `MY-SECRET` is the name of the secret you want to use with your credentials.

VMware recommends using a set of credentials with the least amount of privilege that allows reading
the signature stored in your registry.

## <a id="verify-configuration"></a> Verify your configuration

A sample policy:

```yaml
apiVersion: policy.sigstore.dev/v1beta1
kind: ClusterImagePolicy
metadata:
  name: image-policy
spec:
  images:
  - glob: "gcr.io/projectsigstore/cosign*"
  authorities:
  - name: official-cosign-key
    key:
      data: |
        -----BEGIN PUBLIC KEY-----
        MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEhyQCx0E9wQWSFI9ULGwy3BuRklnt
        IqozONbbdbqz11hlRJy9c7SG+hdcFl9jE9uE/dwtuwU2MqU9T/cN0YkWww==
        -----END PUBLIC KEY-----
```

When using the sample policy, run these commands to verify your configuration:

1. Verify that the Policy Controller admits the signed image that validates with the configured
   public key by running:

   ```console
   kubectl run cosign \
     --image=gcr.io/projectsigstore/cosign:v1.2.1 \
     --dry-run=server
   ```

   For example:

   ```console
   $ kubectl run cosign \
     --image=gcr.io/projectsigstore/cosign:v1.2.1 \
     --dry-run=server
   pod/cosign created (server dry run)
   ```

   If you are using vSphere IaaS control plane (formerly named vSphere with Tanzu) or OpenShift, you
   must add some overrides as in this example:

   ```console
   $ kubectl run cosign \
     --image=gcr.io/projectsigstore/cosign:v1.2.1 \
     --overrides='{"spec": {"securityContext": {"seccompProfile": {"type": "RuntimeDefault"}}, "containers": [{"name": "cosign", "securityContext": {"allowPrivilegeEscalation": false, "runAsNonRoot": true, "capabilities": {"drop": ["ALL"]}}}]}}' \
     --override-type strategic \
     --dry-run=server
   pod/cosign created (server dry run)
   ```

1. Verify that the Policy Controller rejects the unmatched image by running:

   ```console
   kubectl run busybox --image=busybox --dry-run=server
   ```

   For example:

   ```console
   $ kubectl run busybox --image=busybox --dry-run=server
     Error from server (BadRequest): admission webhook "policy.sigstore.dev" denied the request: validation failed: no matching policies: spec.containers[0].image
     index.docker.io/library/busybox@sha256:3614ca5eacf0a3a1bcc361c939202a974b4902b9334ff36eb29ffe9011aaad83
   ```

   In the example output, it did not specify which authorities were used as there was no policy
   found that matched the image. Therefore the image fails to validate for a signature and fails to
   deploy.

1. Verify that the Policy Controller rejects a matched image signed with a different key than the
   one configured by running:

   ```console
   kubectl run cosign-fail \
     --image=gcr.io/projectsigstore/cosign:v0.3.0 \
     --dry-run=server
   ```

   For example:

   ```console
   $ kubectl run cosign-fail \
       --image=gcr.io/projectsigstore/cosign:v0.3.0 \
       --dry-run=server
     Error from server (BadRequest): admission webhook "policy.sigstore.dev" denied the request: validation failed: failed policy: image-policy: spec.containers[0].image
     gcr.io/projectsigstore/cosign@sha256:135d8c5e27bdc917f04b415fc947d7d5b1137f99bb8fa00bffc3eca1856e9c52 failed to validate public keys with authority official-cosign-key for gcr.io/projectsigstore/cosign@sha256:135d8c5e27bdc917f04b415fc947d7d5b1137f99bb8fa00bffc3eca1856e9c52: no matching signatures:
   ```

   In the example output, it specifies which authorities were used for validation when a policy was
   found that matched the image. In this case, the authority used was `official-cosign-key`. If no
   name is specified, it is `authority-#` by default.
