# Run queries with GraphQL

This topic tells you how to connect to the GraphQL playground and how to run some queries.

## <a id='connecting-to-graphql'></a> Connect to Artifact Metadata Repository (AMR) GraphQL

You can perform GraphQL queries by using the GraphQL playground or by using [curl](https://curl.se/).

Use the GraphQL playground
: Complete the following steps to perform GraphQL queries:

  1. Enable ingress. VMware recommends enabling ingress. The Supply Chain Security Tools (SCST) –
     Store and AMR packages share the same ingress configuration. For information about enabling
     ingress, see [Ingress support for Supply Chain Security Tools - Store](../ingress.hbs.md).
  1. Retrieve the access token by running:

     ```console
     kubectl -n metadata-store get secret amr-graphql-view-token -o json | jq -r ".data.token" | base64 -d
     ```

  1. To connect to the AMR GraphQL playground, visit `https://amr-graphql.INGRESS-DOMAIN/play`,
     where `INGRESS-DOMAIN` is the domain of the ingress you want to use.

  1. In the **Headers** tab at the bottom of the query window, add a JSON block containing the
     following authentication header:

     ```json
     {
       "Authorization": "Bearer ACCESS-TOKEN"
     }
     ```

     Where `ACCESS-TOKEN` is the AMR GraphQL access token. You can use this to write and run your
     own GraphQL queries to fetch data from the AMR.

Use curl
: Write and run GraphQL queries to fetch data from the AMR. This procedure uses curl to query
  the AMR GraphQL endpoint, but you can use other similar tools to access the endpoint:

  1. Enable ingress. VMware recommends enabling ingress. The SCST – Store and AMR packages share the
     same ingress configuration. For information about enabling ingress, see
     [Ingress support for SCST - Store](../ingress.hbs.md).
  1. To connect to the AMR GraphQL by using curl after you enable ingress, you must first get the
     AMR GraphQL access token and its certificate authority (CA) certificate by running:

     ```console
     kubectl get secret ingress-cert -n metadata-store -o json | jq -r '.data."ca.crt"' | base64 -d > /tmp/graphql-ca.crt
     ```

  1. After the token and certificate are retrieved, use curl to perform GraphQL queries by running:

     ```console
     curl "https://amr-graphql.INGRESS-DOMAIN/query" \
       --cacert FILE-LOCATION \
       -H "Authorization: Bearer ACCESS-TOKEN" \
       -H 'accept: application/json' \
       -H 'content-type: application/json' \
       --data-raw '{"query":"query getAppAcceleratorRuns { appAcceleratorRuns(first: 250){ nodes { guid name namespace timestamp } pageInfo{ endCursor hasNextPage } } }"}' | jq .
     ```

     Where:

     - `INGRESS-DOMAIN` is the domain of the ingress you want to use.
     - `ACCESS-TOKEN` is the AMR GraphQL access token.
     - `FILE-LOCATION` is the file containing the AMR GraphQL CA certificate. For example,
       `/tmp/graphql-ca.crt`.

## <a id='query-app-accel-runs'></a> Query for AppAcceleratorRuns (alpha)

This section tells you about GraphQL query arguments, and lists the fields available for
`AppAcceleratorRuns` and `AppAcceleratorFragments`.

### <a id='app-accel-query-args'></a> AppAcceleratorRuns query arguments

You can specify the following arguments when querying for `AppAcceleratorRuns`. If you do not
specify an argument the query returns all `AppAcceleratorRuns`. `query` expects an object that
specifies additional arguments to query.

| Argument | Description | Example |
|----------|-------------|---------|
|`guid`|String value unique identifier for each `AppAcceleratorRuns`.|`appAcceleratorRuns(query:{guid: "d2934b09-5d4c-45da-8eb1-e464f218454e"})`|
|`source`|String representing the client used to run the accelerator. Supported values include `TAP-GUI`, `VSCODE`, and `INTELLIJ`.|`appAcceleratorRuns(query:{source: "TAP-GUI"})`|
|`username`|String representing the user name of the person who runs the accelerator, as captured by the client UI.|`appAcceleratorRuns(query:{username: "test.user"})`|
|`namespace` and `name`|Strings representing the accelerator that was used to create an application.|`appAcceleratorRuns(query:{name: "tanzu-java-web-app"})`|
|`appAcceleratorRepoURL`, `appAcceleratorRevision`, and `appAcceleratorSubpath`|Location in VCS (Version Control System) of the accelerator sources used.|`appAcceleratorRuns(query:{appAcceleratorRepoURL: "https://github.com/vmware-tanzu/application-accelerator-samples.git", appAcceleratorRevision: "v1.6"`|
|`timestamp`|String representation of the time the accelerator ran. You can query for runs that happened `before` or `after` a particular instant.|`appAcceleratorRuns(query: {timestamp: {after: "2023-10-11T13:40:46.952Z"}}`)|

### <a id='app-accel-runs-fields'></a> AppAcceleratorRuns fields

You can choose the following fields to return in the GraphQL query. You must specify at least one
field.

- `guid`: String value unique identifier for each `AppAcceleratorRuns`.
- `source`: String representing the client used to run the accelerator.
- `username`: String representing the user name of the person who ran the accelerator.
- `namespace` and `name`: Strings representing the accelerator that was used to create an
  application.
- `appAcceleratorRepoURL`, `appAcceleratorRevision`, and `appAcceleratorSubpath`: Location in VCS of
  the accelerator sources used.
- `timestamp`: String representation of the time the accelerator ran.
- `appAcceleratorSource`: VCS information of the sources of the accelerator used, but navigable as a
  commit.
- `appAcceleratorFragments`: A one-to-many container of nodes representing the fragment versions
  used in each `AppAcceleratorRuns`. Fragment nodes share many of the fields with
  `AppAcceleratorRuns`, with the same semantics but applied to the particular fragment. These
  include:
  - `namespace` and `name`: Strings representing the identity of the fragment.
  - `appAcceleratorFragmentSourceRepoURL`, `appAcceleratorFragmentSourceRevision`, and
    `appAcceleratorFragmentSourceSubpath`: Location in VCS of the sources of the fragment used.
  - `appAcceleratorFragmentSource`: VCS information of the sources of the fragment, but navigable as
    a commit.

### <a id='sample-app-accel-query'></a> Sample Application Accelerator queries

- Get the list of all Application Accelerator runs, with the fragments used for each, with this
  example:

  ```graphql
    query getAllAcceleratorRuns {
      appAcceleratorRuns {
        nodes {
          name
          appAcceleratorFragments {
            nodes {
              name
            }
          }
        }
      }
    }
  ```
