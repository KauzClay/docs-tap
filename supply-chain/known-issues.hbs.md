# Tanzu Supply Chain known issues

This topic lists known issues for Tanzu Supply Chain.

{{> 'partials/supply-chain/beta-banner' }}

## Component authoring

- Components cannot have more than one resumption defined. When there are multiple resumptions, the
  `WorkloadRuns` are not correctly created after changes triggered by these resumptions. The
  current workaround is to assess all triggers in a single resumption.

## Kubernetes distribution support

- The beta release of Tanzu Supply Chain does not include support for Red Hat OpenShift. However,
  this support is planned for an upcoming release. Attempting to individually install components for
  Tanzu Supply Chains and Managed Resource Controller, as well as installing the Authoring profile
  that includes those components as standard, results in failure due to the absence of this
  support.

## CA certificate support

- The beta version of Tanzu Supply Chain currently does not include support for CA certificates in
  the out-of-the-box components. However, you can edit the components to support CA certificates and
  use them to construct a new Supply Chain. Support for CA certificates as standard is planned for
  future versions.