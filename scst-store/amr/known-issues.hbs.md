# Known Issues - Artifact Metadata Repository Observer

This topic tells you about known issues with Artifact Metadata Repository (AMR) Observer.

## <a id='known-issue-tmc'></a> Tanzu Mission Control

When `observer.deploy_through_tmc` is `true`, properties are auto-configured for Tanzu Mission
Control (TMC). This causes the `MultiClusterPropertyCollector` resource to overwrite existing Tanzu
Application Platform values for Observer.

### <a id='ki-tmc-acme-issuer'></a> Let's Encrypt ACME self-signed issuer

When using Let's Encrypt ACME self-signed issuers, the resultant Kubernetes secret resource does not
contain a `ca.crt` property. Therefore, when the `MultiClusterPropertyCollector` resource creates
the Observer package configuration values secret, the required `ca_cert_data` is empty.

To work around this issue, add the Certificate Authority (CA) Certificate to the
`shared.ca_cert_data` key in the Tanzu Application Platform installation values.