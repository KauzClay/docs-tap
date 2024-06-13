# Troubleshoot Crossplane

This topic explains how you troubleshoot issues related to Crossplane on Tanzu Application Platform
(commonly known as TAP).

For the limitations of Crossplane, see [Crossplane limitations](../reference/known-limitations.hbs.md).

## <a id=“resource-already-exists”></a> Resource already exists error when installing Crossplane

**Symptom:**

Installation of Crossplane, or a Tanzu Application Platform profile that includes Crossplane, fails
with the error:

```console
Resource already exists
```

**Explanation:**

Crossplane is already installed on the cluster. You cannot install the Crossplane package on a cluster
that already has Crossplane installed on it by using another method, such as Helm install.

**Solution:**

Exclude the Crossplane package in the `tap-values.yaml` file.
For more information, see [Use your existing Crossplane installation](./use-existing-crossplane.hbs.md).

---

## <a id="validatingwebhookconfig"></a>The `validatingwebhookconfiguration` is not removed when you uninstall the Crossplane Package

**Symptom:**

The Crossplane Package deploys a `validatingwebhookconfiguration` named `crossplane` during installation.
This resource is not deleted when you uninstall the Package.

**Solution:**

Delete the `validatingwebhookconfiguration` manually by running:

```console
kubectl delete validatingwebhookconfiguration crossplane
```

---

## <a id="tls-verification-error"></a> Claims never transition to `READY=True`

**Symptom:**

On rare occasions, service claims you create do not transition to `READY=True`.
If you inspect the underlying Crossplane managed resource, you find a TLS certificate verification
error similar to the following:

```console
Warning  ComposeResources   39s (x23 over 17m)  defined/compositeresourcedefinition.apiextensions.crossplane.io  cannot compose
resources: cannot run Composition pipeline step "patch-and-transform": cannot run Function "function-patch-and-transform": rpc error:
code = Unavailable desc = last connection error: connection error: desc = "transport: authentication handshake failed: tls: failed to
verify certificate: x509: certificate signed by unknown authority (possibly because of \"crypto/rsa: verification error\" while trying
to verify candidate authority certificate \"Crossplane\")"
```

**Explanation:**

This issue occurs due to the way Crossplane manages the life cycle of various TLS certificates, in particular,
the root CA certificate found in the `crossplane-root-ca` secret in the `crossplane-system` namespace.
This certificate signs all of the other certificates that Crossplane uses.

Occasionally, this certificate can get corrupted during the Crossplane installation.
This behavior is described in [Function: certificate signed by unknown authority "Crossplane" #5456](https://github.com/crossplane/crossplane/issues/5456) in GitHub.

**Solution:**

As a workaround:

1. Delete all secrets in the `crossplane-system` namespace.
1. Recreate all pods in the `crossplane-system` namespace by deleting the existing ones.

---

## <a id="upgrade-failing-reconcile"></a>Crossplane component failing to reconcile during upgrade

**Symptom:**

The Crossplane package fails to reconcile when upgrading the package. The providers and
`function-patch-and-transform` fail with error message:

```console
cannot resolve package dependencies: missing node in tree
```

You can see the error message if you run these commands:

```console
kubectl get provider.pkg -n crossplane-system -o yaml
kubectl get functions -n crossplane-system -o yaml
```

**Explanation:**

When upgrading the Crossplane package, this issue can occur if the source of the package has changed
without changing its version.

**Solution:**

As a workaround:

1. Edit the `lock` and update the package source for `provider-helm`, `provider-kubernetes`, and
   `function-patch-and-transform` with the new package source as follows:

    ```console
    kubectl edit locks.pkg.crossplane.io lock

    packages:
    - dependencies: []
      name: provider-helm-919772e449f3
      source: tanzu.packages.broadcom.com/tanzu-application-platform/tap-packages:provider-helm
      type: Provider
      version: ...
    - dependencies: []
      name: function-patch-and-transform-7799bf5e4e7f
      source: tanzu.packages.broadcom.com/tanzu-application-platform/tap-packages:function-patch-and-transform
      type: Function
      version: ...
    - dependencies: []
      name: provider-kubernetes-28e95f897554
      source: tanzu.packages.broadcom.com/tanzu-application-platform/tap-packages:provider-kubernetes
      type: Provider
      version: ...
    ```

1. Verify that the providers and functions are healthy, and that Crossplane package has reconciled by running:

    ```console
    kubectl get provider.pkg -n crossplane-system
    kubectl get functions -n crossplane-system
    kubectl get packageinstall crossplane -n tap-install
    ```
