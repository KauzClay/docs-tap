# ClientRegistration

`ClientRegistration` is the request for client credentials for an [`AuthServer`](./authserver.md).

It implements the [Service Bindings'](https://servicebinding.io/spec/core/1.0.0/) `ProvisionedService`. The credentials
are returned as a [Service Bindings](https://servicebinding.io/spec/core/1.0.0/) `Secret`.

A `ClientRegistration` needs to uniquely identify an `AuthServer` via `spec.authServerSelector`. If it matches none,
too many or a disallowed `AuthServer` it won't get credentials. The other fields are for the configuration of the
client on the `AuthServer`.

## Spec

```yaml
apiVersion: sso.apps.tanzu.vmware.com/v1alpha1
kind: ClientRegistration
metadata:
  name: ""
  namespace: ""
spec:
  authServerSelector: # required
    matchLabels: { }
  redirectURIs: # required
    - ""
  scopes: # optional
    - name: ""
      description: ""
  authorizationGrantTypes: # optional
    - client_credentials
    - authorization_code
    - refresh_token
  clientAuthenticationMethod: basic # or "post", optional
  requireUserConsent: false # optional
status:
  authServerRef:
    apiVersion: ""
    issuerURI: ""
    kind: ""
    name: ""
    namespace: ""
  binding:
    name: ""
  clientID: ""
  clientSecretHelp: ""
  conditions:
    - lastTransitionTime: ""
      message: ""
      reason: ""
      status: "True" # or "False"
      type: ""
  observedGeneration: 0
```

Alternatively, you can interactively discover the spec with:

```shell
kubectl explain clientregistrations.sso.apps.tanzu.vmware.com
```

## Status & conditions

The `.status` subresource helps you to learn about your client credentials, the matched `AuthServer` and to troubleshoot
issues.

`.status.authServerRef` identifies the successfully matched `AuthServer` and its issuer URI.

`.status.binding.name` is the name of the Service Bindings `Secret` which contains the client credentials.

`.status.conditions` documents each step in the reconciliation:

- `Valid`: Is the spec valid?
- `AuthServerResolved`: Has the targeted `AuthServer` been resolved?
- `ClientSecretResolved`: Has the client secret been resolved?
- `ServiceBindingSecretApplied`: Has the Service Bindings Secret with the client credentials been applied?
- `AuthServerConfigured`: Has the resolved `AuthServer` been configured with the client?
- `Ready`: whether all the previous conditions are "True"

The super condition `Ready` denotes a fully successful reconciliation of a given `ClientRegistration`.

If everything goes well you will see something like this:

```yaml
status:
  authServerRef:
    apiVersion: sso.apps.tanzu.vmware.com/v1alpha1
    issuerURI: http://authserver-sample.default
    kind: AuthServer
    name: authserver-sample
    namespace: default
  binding:
    name: clientregistration-sample
  clientID: default_clientregistration-sample
  clientSecretHelp: 'Find your clientSecret: ''kubectl get secret clientregistration-sample --namespace default'''
  conditions:
    - lastTransitionTime: "2022-05-13T07:56:41Z"
      message: ""
      reason: Updated
      status: "True"
      type: AuthServerConfigured
    - lastTransitionTime: "2022-05-13T07:56:40Z"
      message: ""
      reason: Resolved
      status: "True"
      type: AuthServerResolved
    - lastTransitionTime: "2022-05-13T07:56:40Z"
      message: ""
      reason: ResolvedFromBindingSecret
      status: "True"
      type: ClientSecretResolved
    - lastTransitionTime: "2022-05-13T07:56:41Z"
      message: ""
      reason: Ready
      status: "True"
      type: Ready
    - lastTransitionTime: "2022-05-13T07:56:40Z"
      message: ""
      reason: Applied
      status: "True"
      type: ServiceBindingSecretApplied
    - lastTransitionTime: "2022-05-13T07:56:40Z"
      message: ""
      reason: Valid
      status: "True"
      type: Valid
  observedGeneration: 1
```

## Example

```yaml
apiVersion: sso.apps.tanzu.vmware.com/v1alpha1
kind: ClientRegistration
metadata:
  name: my-client-registration
  namespace: app-team
spec:
  authServerSelector:
    matchLabels:
      for: app-team
      ldap: "true"
  redirectURIs:
    - "https://127.0.0.1:8080/authorized"
    - "https://my-application.com/authorized"
  requireUserConsent: false
  clientAuthenticationMethod: basic
  authorizationGrantTypes:
    - "client_credentials"
    - "refresh_token"
  scopes:
    - name: "openid"
      description: "To indicate that the application intends to use OIDC to verify the user's identity"
    - name: "email"
      description: "The user's email"
    - name: "profile"
      description: "The user's profile information"
```

The client is being registered with the authorization server with the given specs. The resulting client credentials are
available in a Secret that's owned by the ClientRegistration.

```yaml
apiVersion: v1
kind: Secret
type: servicebinding.io/oauth2
data: # fields below are base64-decoded for display purposes only
  type: oauth2
  provider: appsso
  client-id: default_my_client_registration
  client-secret: c2VjcmV0 # auto-generated
  issuer-uri: https://appsso.example.com
  client-authentication-method: basic
  scope: openid,email,profile
  authorization-grant-types: client_credentials,refresh_token
```
