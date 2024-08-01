# Edit your AWS RDS PostgreSQL configuration

This topic describes how you can edit your AWS RDS PostgreSQL configuration for Supply Chain
Security Tools (SCST) - Store.

## <a id='prereq'></a> Before you begin

You must have an AWS account.

## <a id='aws-rds'></a> Set up a certificate and configuration

To set up a certificate and configuration:

1. Create an Amazon RDS PostgreSQL DB instance by using the
   [Amazon RDS documentation](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.PostgreSQL.html#CHAP_GettingStarted.Creating.PostgreSQL)

2. After the database instance starts, retrieve the following information:

   - Database Instance Endpoint
   - Master Username
   - Master Password
   - Database Name

   > **Note** If the database name is `-` in the AWS RDS UI, the value is likely to be `postgres`.

3. Create a security group to allow inbound connections from the cluster to the PostgreSQL DB.

4. Retrieve the corresponding CA Certificate that signed the PostgreSQL TLS Certificate by using the
   [Amazon RDS documentation](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.SSL.html)

5. In `metadata-store-values.yaml` replace the following placeholders with your values:

    ```yaml
    db_host: "<DB Instance Endpoint>"
    db_user: "<Master Username>"
    db_password: "<Master Password>"
    db_name: "<Database Name>"
    db_port: "5432"
    db_sslmode: "verify-full"
    db_max_open_conns: 10
    db_max_idle_conns: 100
    db_conn_max_lifetime: 60
    db_ca_certificate: |
       <Corresponding CA Certification>
       ...
       ...
       ...
    deploy_internal_db: "false"
    ```

> **Note** If `deploy_internal_db` is set to `false,` an instance of PostgreSQL is not deployed in
> the cluster.
