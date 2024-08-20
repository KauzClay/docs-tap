# Output types for catalog components

This topic describes the common types established in Tanzu Supply Chain components, their purpose,
and their structure. Use this as a guide to consuming or emitting types from your own custom
components.

{{> 'partials/supply-chain/beta-banner' }}

## conventions

- **URL**: OCI image containing the `app-config.yaml` file
- **Digest**: OCI image digest

This output image contains a single file, `app-config.yaml`. The `template` field in this file
contains a Kubernetes Pod Template specification, decorated by the Conventions controller. The
Conventions component can also copy the settings in the configurations `spec.env` into the Pod
template's `env` field.

## git

- **URL**: The source code Git repository URL
- **Digest**: The source code git commit SHA

## git-pr

- **URL**: URL of the pull request
- **Digest**: SHA digest of the pull request commit

This is the output for a Git pull request.

## image

- **URL**: OCI image URL for the image built with kpack
- **Digest**: OCI image digest

This is an OCI image containing the app built with kpack.

## oci-yaml-files

- **URL**: An OCI image containing Kubernetes resource(s) as YAML files
- **Digest**: OCI image digest

The resources in this OCI image can be applied to a cluster with no modifications. The purpose of
this OCI image type is to provide the raw Kubernetes resources required to deploy an application on
to a cluster.

## oci-ytt-files

- **URL**: An OCI image containing files written in the [ytt](https://carvel.dev/ytt) templating
  language.
- **Digest**: OCI image digest

The files in this OCI image cannot be applied to a cluster, and are intended to be combined with
Kubernetes resources from `oci-yaml-files`.

The purpose of this OCI image type is to provide the ytt values schema and overlays that make a
Carvel package configurable. Separating the ytt files from the raw Kubernetes resources makes the
`app-config-*` components more generally useful.

If a Supply Chain author wants to create a Supply Chain that simply deploys the raw Kubernetes YAML
files created by the `app-config-*` components, instead of putting them into a Carvel Package, the
author can write a component that takes the input `oci-yaml-files`, and not the input
`oci-ytt-files`.

## package

- **URL**: An OCI image containing a Carvel package, `PackageInstall`, and values YAML file
- **Digest**: OCI image digest

The resources in this OCI image can either be deployed to a cluster or written to a Git repository
for use in a GitOps workflow.

The purpose of this OCI image type is to contain a single version of an application as a Carvel
package, as well as the `PackageInstall` and values YAML necessary to deploy the package.

## source

- **URL**: OCI image containing the Git repository source code
- **Digest**: OCI image digest

This is an OCI image containing the source code retrieved from a Git repository.
