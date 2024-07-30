# Retrieve access tokens for Supply Chain Security Tools - Store

This topic describes how you can retrieve access tokens for Supply Chain Security Tools (SCST) -
Store.

## Overview

When you install Tanzu Application Platform, the SCST - Store deployment automatically includes a
read-write service account. This service account is bound to the `metadata-store-read-write` role.

There are two types of SCST - Store service accounts:

- The read-write service account, which is for full access to the `POST` and `GET` API requests
- The read-only service account, which can only use `GET` API requests

This topic shows how to retrieve the access token for these service accounts.

## Retrieving the read-write access token

To retrieve the read-write access token, run:

```console
kubectl get secrets metadata-store-read-write-client -n metadata-store -o jsonpath="{.data.token}" | base64 -d
```

## Retrieving the read-only access token

To retrieve the read-only access token, you must first
[create read-only service account](create-service-account.hbs.md#ro-serv-accts).

To retrieve the read-only access token, run:

```console
kubectl get secrets metadata-store-read-client -n metadata-store -o jsonpath="{.data.token}" | base64 -d
```

## Using an access token

The access token is a Bearer token used in the HTTP request header `Authorization`. For example,
`Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IjhMV0...`.

## Additional Resources

- [Create service accounts](create-service-account.hbs.md)
- [Create a service account with a custom cluster role](custom-role.hbs.md)
