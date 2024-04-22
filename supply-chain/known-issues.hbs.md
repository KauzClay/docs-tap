# Tanzu Supply Chains Known Issues

This topic lists known issues for Tanzu Supply Chains.

{{> 'partials/supply-chain/beta-banner' }}

## Component Authoring

- Components cannot have more than one resumption defined. When there are multiple resumptions,
the `WorkloadRuns` are not being correctly created upon changes triggered by these resumptions.
The current workaround is to assess all triggers in a single resumption.

## Kubernetes Distribution Support

- The Beta release of Tanzu Supply Chain does not include support for Red Hat OpenShift. However, this support is planned for an upcoming release. Attempting to individually
install components for Tanzu Supply Chains and Managed Resource Controller, as well as installing
the Authoring profile that comes with those components out of the box, results in failure due
to the absence of this support.

## CA Cert Support

- The Beta version of Tanzu Supply Chain currently does not include support for CA certificates in the Out of the Box components. However, users can modify the components to support CA certificates and utilize them to construct a new Supply Chain. The support for CA certificates out of the box is part of the roadmap and will be included in future versions.