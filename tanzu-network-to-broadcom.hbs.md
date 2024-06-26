# Migrate Tanzu Application Platform images from VMware Tanzu Network

This topic tells you how to update your Tanzu Application Platform (commonly known as TAP)
installation to use images located in your own registry instead of VMware Tanzu Network registry.

Tanzu Application Platform releases have migrated from VMware Tanzu Network to the Broadcom Support Portal
and Broadcom registry.
Using VMware Tanzu Network to install or upgrade Tanzu Application Platform is no longer supported.
It is now mandatory to relocate the Tanzu Application Platform images from `tanzu.packages.broadcom.com`
to your own container image registry.

## <a id="who-is-this-for"></a> Who must follow these steps

You must follow the procedures in this topic if both of the following conditions apply:

- The Tanzu Application Platform images for your installation are located in `registry.tanzu.vmware.com`
  instead of in your own registry because you skipped the install step
  [Relocate images to a registry](install-online/profile.hbs.md#relocate-images).

- You are not upgrading your Tanzu Application Platform installation. If you are upgrading, you can
  relocate the Tanzu Application Platform images as part of the steps in
  [Upgrade Tanzu Application Platform](upgrading.hbs.md).

## <a id="migrate"></a> Migrate Tanzu Application Platform images

To update your Tanzu Application Platform installation to use images located in your own registry
instead of VMware Tanzu Network registry, you must:

- [Update the package repository](#update-package-repo)
- [Restart the kapp controller](#restart-kapp-contoller)

### <a id="update-package-repo"></a> Update the package repository

Follow these steps to update the package repository:

1. Relocate the Tanzu Application Platform images by following step 1 through step 6
  in [Relocate images to a registry](install-online/profile.hbs.md#relocate-images).

    > **Important** You must set the `TAP_VERSION` environment variable to the same version of
    > Tanzu Application Platform that you have already installed, for example, `{{ vars.tap_version }}`.

1. Update the source URL of the package repository to the container image registry where you relocated
   the Tanzu Application Platform images to by running:

    ```console
    tanzu package repository update tanzu-tap-repository \
    --url ${INSTALL_REGISTRY_HOSTNAME}/TARGET-REPOSITORY/tap-packages:${TAP_VERSION} \
    --namespace tap-install
    ```

    Where `TARGET-REPOSITORY` is your target repository. This is a folder or repository on in your registry
    that serves as the location for the Tanzu Application Platform installation files.

1. Verify that you added the new package repository by running:

    ```console
    tanzu package repository get TAP-REPO-NAME --namespace tap-install
    ```

    Where `TAP-REPO-NAME` is the package repository name. It must match with `tanzu-tap-repository`
    in the previous step.

### <a id="restart-kapp-contoller"></a> Restart the kapp controller

Restart the kapp controller so that installed applications can pull the images from your
container image registry. Run:

```console
kubectl rollout restart deployment kapp-controller -n kapp-controller
```

Wait for 10 to 15 mins for the installed apps to start pulling images from your container image registry.
