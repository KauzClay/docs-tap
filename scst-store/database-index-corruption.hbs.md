# Supply Chain Security Tools - Store Database Index Corruption

This topic tells you how to troubleshoot the Metadata Store database index issue after upgrading
from v1.5 to a later version of Supply Chain Security Tools (SCST) - Store.

## <a id='overview'></a> Overview

The index corruption happened because of a base OS upgrade in the PostgreSQL database images to
address some of the CVE which includes a breaking change in `glibc locale` library.

- In Tanzu Application Platform v1.5 and earlier, `Postgres-bionic-13:1.22.0` uses `ubuntu:18.04`,
  which includes `ubuntu/glibc 2.27-3ubuntu1.6`.
- In Tanzu Application Platform v1.6 and later, `Postgres-bionic-13:1.23.0` uses `ubuntu:22.04` and
  `jammy-20221101`, which includes `ubuntu/glibc 2.35-0ubuntu3.1`.

`glibc v2.28` includes a major update for the locale data, which might corrupt indices after
upgrading. For more information, see the
[PostgreSQL wiki](https://wiki.postgresql.org/wiki/Locale_data_changes).

## <a id='db-index-corrupt'></a> Solve a database index corruption issue reported in Metadata Store App Container logs

The following sections tell you how to detect and solve a database index corruption issue reported
in Metadata Store App Container logs.

### <a id='db-index-corrupt-symptom'></a> Symptom

The Metadata Store cannot reconcile because the Metadata Store pod reports a potential database
index corruption issue.

```console
kubectl logs metadata-store-app-pod_name -n metadata-store
```

#### <a id='db-index-corrupt-output'></a> Output

```console
{“level”:“error”,“ts”:“2023-08-15T16:38:31.528115988Z”,“logger”:“MetadataStore”,“msg”:“unable to check index corruption since user is not a superuser to perform \“CREATE EXTENSION amcheck\“. Please create this extension and check for index corruption using following sql command \“SELECT bt_index_check(oid) FROM pg_class WHERE relname in (SELECT indexrelid::regclass::text FROM (SELECT indexrelid, indrelid, indcollation[i] coll FROM pg_index, generate_subscripts(indcollation, 1) g(i)) s JOIN pg_collation c ON coll=c.oid WHERE collprovider IN (‘d’, ‘c’) AND collname NOT IN (‘C’, ‘POSIX’));\“”,“hostname”:“metadata-store-app-77c9fb59c8-qplxt”}
{“level”:“error”,“ts”:“2023-08-15T16:38:31.528139637Z”,“logger”:“MetadataStore”,“msg”:“Found corrupted database indexes but unable to fix them”,“hostname”:“metadata-store-app-77c9fb59c8-qplxt”,“error”:“unable to check index corruption since user is not a superuser to perform \“CREATE EXTENSION amcheck\“. Please create this extension and check for index corruption using following sql command \“SELECT bt_index_check(oid) FROM pg_class WHERE relname in (SELECT indexrelid::regclass::text FROM (SELECT indexrelid, indrelid, indcollation[i] coll FROM pg_index, generate_subscripts(indcollation, 1) g(i)) s JOIN pg_collation c ON coll=c.oid WHERE collprovider IN (‘d’, ‘c’) AND collname NOT IN (‘C’, ‘POSIX’));\“”}
```

### <a id='db-index-corrupt-solution'></a> Solutions

You can solve this issue by upgrading Tanzu Application Platform with updated schema values to
connect to the PostgreSQL database directly and then fix the index manually.

The rest of this topic describes alternative solutions:

- Edit the `auto_correct_db_indexes` property to be `true` in `tap-values.yaml`
- Connect to the Metadata Store database with the account that the Metadata Store API uses
- Connect to the Metadata Store database with a superuser account and manually fix the index

#### Edit the `auto_correct_db_indexes` property to be `true` in `tap-values.yaml`

To edit the `auto_correct_db_indexes` property:

1. Change the property `auto_correct_db_indexes` value to `true` in `tap-values.yaml`.

1. Update Tanzu Application Platform by running:

   ```console
   tanzu package installed update tap -p tap.tanzu.vmware.com -v \{{ vars.tap_version }}  \
   --values-file tap-values.yaml -n tap-install
   ```

1. Examine the Metadata Store API logs and verify that there are no errors.

#### Connect to the Metadata Store database with the account that the Metadata Store API uses

To connect to the Metadata Store database with the same account that the Metadata Store API uses:

1. [Connect to the PostgreSQL database](connect-to-database.hbs.md).
1. Run this SQL to re-index individual tables:

   ```console
   REINDEX TABLE  "artifact_groups";
   REINDEX TABLE  "artifact_group_images";
   REINDEX TABLE  "images";
   REINDEX TABLE  "sources";
   REINDEX TABLE  "source_images";
   REINDEX TABLE  "packages";
   REINDEX TABLE  "image_packages";
   REINDEX TABLE  "vulnerabilities";
   REINDEX TABLE  "package_vulnerabilities";
   REINDEX TABLE  "source_packages";
   REINDEX TABLE  "artifact_group_sources";
   REINDEX TABLE  "method_types";
   REINDEX TABLE  "ratings";
   REINDEX TABLE  "package_managers";
   REINDEX TABLE  "artifact_group_labels";
   REINDEX TABLE  "analysis_instances";
   REINDEX TABLE  "vulnerability_analyses";
   REINDEX TABLE  "reports";
   REINDEX TABLE  "report_ratings";
   REINDEX TABLE  "report_packages_vulnerabilities";
   REINDEX TABLE  "index_migration_statuses";
   ```

1. Run the following SQL:

   ```console
   INSERT INTO index_migration_statuses (created_at, updated_at, "version", status) VALUES(now(), \
   now(), version(), true)
   ```

1. Examine the Metadata Store API logs and verify that there are no errors.

#### Connect to the Metadata Store database with a superuser account and manually fix the index

To connect to the Metadata Store database with a superuser account and manually fix the index:

1. [Connect to the PostgreSQL database](connect-to-database.hbs.md).
1. Install the extension on your PostgreSQL database.

   ```console
   CREATE EXTENSION amcheck
   ```

1. Run the following SQL:

   ```console
   SELECT indexrelid::regclass::text FROM (SELECT indexrelid, indrelid, indcollation[i] coll \
   FROM pg_index, generate_subscripts(indcollation, 1) g(i)) s JOIN pg_collation c ON coll=c.oid
   WHERE collprovider IN ('d', 'c') AND collname NOT IN ('C', 'POSIX');
   ```

1. Run the following SQL when you see an error:

   ```console
   SELECT indexrelid::regclass::text FROM (SELECT indexrelid, indrelid, indcollation[i] coll \
   FROM pg_index, generate_subscripts(indcollation, 1) g(i)) s JOIN pg_collation c ON coll=c.oid
   WHERE collprovider IN ('d', 'c') AND collname NOT IN ('C', 'POSIX');
   ```

1. Run the following SQL if the previous step caused an error:

   ```console
   REINDEX database "metadata-store"
   ```

1. Run the following SQL:

   ```console
   INSERT INTO index_migration_statuses (created_at, updated_at, "version", status) VALUES(now(), \
   now(), version(), true)
   ```

1. Examine the Metadata Store API logs and verify that there are no errors.