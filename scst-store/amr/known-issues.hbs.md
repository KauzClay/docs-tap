# Known Issues - AMR Observer

## <a id='known-issue-tmc'></a> Tanzu Mission Control

With the `observer.deploy_through_tmc` set to `true`, properties are 
auto-configured for TMC. A consequence of this results in existing TAP Values 
for Observer to be overwritten by the `MultiClusterPropertyCollector` resource.

### <a id='known-issue-tmc-acme-issuer'> Let's Encrypt ACME Self Signed Issuer

When using Let's Encrypt ACME Self Signed Issuers, the resultant Kubernetes
secret resource does not contain a `ca.crt` property. Therefore, when the 
`MultiClusterPropertyCollector` resource creates the Observer package 
configuration values secret, the required `ca_cert_data` will be empty.

To workaround this issue, the CA Certificate can be added to the 
`shared.ca_cert_data` key in the TAP installation values.