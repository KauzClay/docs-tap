# Use an external PostgreSQL database for Supply Chain Security Tools - Store

This topic tells you how to configure Tanzu Application Platform (commonly known as TAP) to use an
external database.

For production deployments, VMware recommends that you use an external PostgreSQL database rather
than the one packaged with Tanzu Application Platform. You can manage an external database by using
the best practices and processes that your organization has established.

## <a id='prereqExtrenalDB'></a> Before you begin

Gather the following connection information for the external PostgreSQL database:

- Database Instance Connect Endpoint
- CA Certificate for the database instance TLS certificate (if applicable)
- Database credentials
- Name of the database

If migrating from the internal database to an external database, back up the data before proceeding.

## Configure Artifact Metadata Repository (AMR) and the Metadata Store (MDS) to use the external database

To configure AMR and MDS to use the external database:

1. Update your `tap-values.yaml` file:

    ```yaml
    metadata_store:
      db_host: "<DB Instance Endpoint>"
      db_user: "<Master Username>"
      db_password: "<Master Password>"
      db_name: "<Database Name>"
      # SEE NOTE BELOW
      # Disable the internal database deployment
      deploy_internal_db: "false"
      db_port: "5432"
      db_max_open_conns: 10
      db_max_idle_conns: 100
      db_conn_max_lifetime: 60
      # If TLS is enabled on PostgreSQL Instance
      db_sslmode: "verify-full"
      # If TLS Certificate is self-signed
      db_ca_certificate: |
        <Corresponding CA Certification>
        ...
        ...
        ...
    ```

   > **Caution** If you initially deployed Tanzu Application Platform with an internal database and
   > are migrating to an external database, setting `deploy_internal_db` to `false` removes the
   > internal instance of PostgreSQL. Back up and migrate your data to the database before setting
   > this value to `false` or you might lose data.

2. Apply the new configuration by running:

   ```console
   tanzu package installed update tap -p tap.tanzu.vmware.com -v {{ vars.tap_version }}  \
   --values-file tap-values.yaml -n tap-install
   ```

3. (Optional) If you are migrating from the internal database to an external database, the
   reconciliation might fail. To manually delete the secret that cert-manager created, and cycle
   the AMR and MDS Services so that the new secret is picked up, run:

   ```console
   kubectl delete secret -n metadata_store postgres-db-tls-cert
   kubectl rollout restart -n metadata_store deployment metadata_store-app
   ```

## Validation

Use Bitnami PostgreSQL to validate. For more information, see the
[Bitnami](https://github.com/bitnami/charts/tree/main/bitnami/postgresql) documentation.