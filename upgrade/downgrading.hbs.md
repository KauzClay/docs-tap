# Downgrade to an earlier Tanzu Application Platform version

This topic tells you how to downgrade your Tanzu Application Platform (commonly known as TAP)
installation to an earlier version.

Downgrading Tanzu Application Platform requires you to add the annotation `packaging.carvel.dev/downgradable: ""`
to the Tanzu Application Platform Package and sub-packages. You can do this using an overlay.
For more information about the `downgradable` annotation, see the
[Carvel Documentation](https://carvel.dev/kapp-controller/docs/v0.32.0/package-consumer-concepts/#downgrading).

To downgrade Tanzu Application Platform:

1. Create an overlay secret, for example:

    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: tap-pkgi-lay-secret
      namespace: tap-install
    stringData:
      overlay-tap-pkgi.yml: |
        #@ load("@ytt:overlay", "overlay")

        #@ def match_packageinstall():
        kind: PackageInstall
        #@ end

        #@overlay/match by=overlay.subset(match_packageinstall()), expects="1+"
        ---
        metadata:
          #@overlay/match missing_ok=True
          annotations:
            #@overlay/match missing_ok=True
            packaging.carvel.dev/downgradable: ""
    ```

1. Add the `downgradable` and overlay secret annotations to the Tanzu Application Platform
   `PackageInstall` resource:

    ```yaml
    apiVersion: packaging.carvel.dev/v1alpha1
    kind: PackageInstall
    metadata:
      name: tap
      namespace: tap-install
      annotations:
        packaging.carvel.dev/downgradable: ""
        ext.packaging.carvel.dev/ytt-paths-from-secret-name.99: tap-pkgi-downgradeable-overlay-secret
    ```

    This adds the `packaging.carvel.dev/downgradable: ""` to all `PackageInstalls` created by
    Tanzu Application Platform through a profile and allows you to downgrade them.

1. Adjust the version in the `packageRepository` to downgrade to the version of Tanzu Application Platform.

    Follow the Tanzu Application Platform upgrade documentation to update the version of
    Tanzu Application Platform to the version that you want. To downgrade, adjust the version down instead of up.

    - [Upgrade Tanzu Application Platform](../upgrading.hbs.md)
    - [Upgrade Tanzu Application Platform by using GitOps](../install-gitops/upgrading.hbs.md)

1. (Optional) If you are using GitOps Reference Implementation (GitOps RI) and External Secret Operator (ESO),
   you can downgrade the ESO version. Add the downgradable annotation to the ESO `PackageInstall`:

    ```yaml
    apiVersion: packaging.carvel.dev/v1alpha1
    kind: PackageInstall
    metadata:
      name: external-secrets.apps.tanzu.vmware.com
      namespace: tanzu-sync
      annotations:
        packaging.carvel.dev/downgradable: ""
    ```

    > **Note** Tanzu Application Platform related packages installed separately, also known as add-ons,
    > each need the `downgradeable` annotation.
    > If you have a kapp app that manages the additional package installs, you can include
    > the preceding overlay secret to the ytt struct of your kapp app.
    > For an example, see this line in [tap-gitops](https://github.com/Mpluya/tap-gitops/blob/9909de7b74d9af1b4bf8ff00d51e88be47ba5b8c/clusters/upgrade-run/cluster-config/config/post-install-app.yaml#L25)
    > in GitHub.
