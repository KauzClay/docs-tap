# Deployment details and configuration for Supply Chain Security Tools - Store

This topic describes how you can deploy and configure your Kubernetes cluster for Supply Chain
Security Tools (SCST) - Store.

## <a id='what-deploy'></a> What is deployed

The installation creates the following in your Kubernetes cluster:

- Four components:

  - `metadata-store` API back end
  - Database
  - AMR API back end. If AMR is deployed, see [Deploying AMR](#amr).
  - AMR CloudEvent Handler. If AMR is deployed, see [Deploying AMR](#amr).

- Services for each of the four components:

  - `metadata-store-app`
  - `metadata-store-db`
  - `amr-cloudevent-handler`. If AMR is deployed, see [Deploying AMR](#amr).
  - `amr-graphql-app`. If AMR is deployed, see [Deploying AMR](#amr).

- A namespace called `metadata-store`.
- Persistent volume claim `postgres-db-pv-claim` in the `metadata-store` namespace.
- A Kubernetes secret in the namespace `tap` is installed to allow pulling SCST - Store images from
  a registry.
- Two ClusterRoles:

  - `metadata-store-read-write-client` is bound to a service account by default, giving the service
    account read and write privileges
  - `metadata-store-read-only` isn't bound to any service accounts, you can bind to it if needed.
    See [Service Accounts](#service-accounts).

- (Optional) An HTTPProxy object for ingress support.

## <a id='configuration'></a> Deployment configuration

All configurations are nested inside `metadata_store` in your Tanzu Application Platform values
deployment YAML. AMR-specific configurations are nested under `amr` in the `metadata_store` section.

### <a id='supported-network'></a> Supported Network Configurations

VMware recommends the following connection methods for Tanzu Application Platform:

- For a single or multicluster configuration with Contour, use `Ingress`.
- For a single cluster without Contour and with `LoadBalancer` support configuration, use
  `LoadBalancer`.
- For a single cluster without Contour and without `LoadBalancer` configuration, use `NodePort`.

Multicluster without Contour configuration is not supported. For a production environment, VMware
recommends installing SCST - Store with ingress enabled.

#### <a id='appserv-type'></a> App service type

The Metadata Store app service type is configured inside the `metadata_store` property of the values
file.

```yaml
metadata_store:
  app_service_type: "ClusterIP"
```

Supported values include:

- `LoadBalancer`
- `ClusterIP`
- `NodePort`

The `app_service_type` is set to `ClusterIP` by default.

#### <a id='ingress'></a> Ingress support

The SCST - Store values file allows you to enable ingress support and to configure a custom domain
name to use Contour to provide external access to the SCST - Store API. These ingress configurations
are shared for the Metadata Store and AMR. Enabling ingress for the Metadata Store enables it for
both the Metadata Store and AMR.

For example:

```yaml
metadata_store:
  ingress_enabled: "true"
  ingress_domain: "example.com"
  app_service_type: "ClusterIP" # recommended setting when ingress is enabled
```

An HTTPProxy object is installed with `metadata-store.example.com` as the FQDN. For more
information, see [Ingress](ingress.hbs.md).

> **Important** The `ingress_enabled` property expects a string value of `"true"` or `"false"`, not
> a Boolean value.

### <a id="db-config"></a> Database configuration

You can start using the Metadata Store with the default database included with the deployment.
The default database deployment does not support many enterprise production requirements, including
scaling, redundancy, and failover. However, it is a secure deployment.

#### <a id='aws-rds-postgresql-data'></a> Use an AWS RDS PostgreSQL database

You can configure the deployment to use your own RDS database instead of the default. For more
information, see [Configure your AWS RDS PostgreSQL configuration](use-aws-rds.hbs.md).

#### <a id='ex-sql-db'></a> Use an external PostgreSQL database

You can configure the deployment to use any other PostgreSQL database. For more
information, see [Use an external PostgreSQL database for SCST - Store](use-external-database.hbs.md).

#### <a id='cust-data-pass'></a> Custom database password

By default, a database password is generated after deployment. To configure a custom password, use
the `metadata_store.db_password` property in the values file.

> **Important** There is a known issue related to changing database passwords. For more information,
> see [Persistent volume retains data](troubleshooting.hbs.md#retains-data).

To configure a custom database password for AMR:

```yaml
metadata_store:
  db_password: "PASSWORD"
```

Where `PASSWORD` is the same password used for both deployments.

### <a id='service-accounts'></a> Service accounts

By default, a service account with read-write privileges to the Metadata Store app is installed.
This service account is a cluster-wide account that uses ClusterRole. If you don't want the service
account and role, set the `add_default_rw_service_account` property to `"false"`. To create a custom
service account, see [Retrieve and create service accounts](create-service-account.hbs.md).

The store creates a read-only cluster role, which is bound to a service account by using
`ClusterRoleBinding`. To create service accounts to bind to this cluster role, see
[Retrieve and create service accounts](create-service-account.hbs.md).

## <a id='export-cert'></a> Export certificates

SCST - Store creates a
[Secret Export](https://github.com/carvel-dev/secretgen-controller/blob/develop/docs/secret-export.md)
for exporting certificates to SCST - Store to securely post scan results. These certificates are
exported to the namespace where SCST - Store is installed.
