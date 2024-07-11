# Example Compositions

This topic gives you example `Composition` resources that you can use to help when integrating cloud
services into Tanzu Application Platform (commonly known as TAP).

## <a id="aws"></a>  AWS RDS example `Composition`

The following `Composition` ensures that all RDS PostgreSQL instances are placed in the `us-east-1` 
region and use the default VPC for the respective AWS account.

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
 labels:
   provider: "aws"
   vpc: "default"
 name: xpostgresqlinstances.bindable.aws.database.example.org
spec:
 compositeTypeRef:
   apiVersion: bindable.database.example.org/v1alpha1
   kind: XPostgreSQLInstance
 publishConnectionDetailsWithStoreConfigRef:
   name: default
 resources:
 - base:
     apiVersion: database.aws.crossplane.io/v1beta1
     kind: RDSInstance
     spec:
       forProvider:
         dbInstanceClass: db.t2.micro
         engine: postgres
         dbName: postgres
         engineVersion: "12"
         masterUsername: masteruser
         publiclyAccessible: true
         region: us-east-1
         skipFinalSnapshotBeforeDeletion: true
       writeConnectionSecretToRef:
         namespace: crossplane-system
   connectionDetails:
   - name: type
     value: postgresql
   - name: provider
     value: aws
   - name: database
     value: postgres
   - fromConnectionSecretKey: username
   - fromConnectionSecretKey: password
   - name: host
     fromConnectionSecretKey: endpoint
   - fromConnectionSecretKey: port
   name: rdsinstance
   patches:
   - fromFieldPath: metadata.uid
     toFieldPath: spec.writeConnectionSecretToRef.name
     transforms:
     - string:
         fmt: '%s-postgresql'
         type: Format
       type: string
     type: FromCompositeFieldPath
   - fromFieldPath: spec.parameters.storageGB
     toFieldPath: spec.forProvider.allocatedStorage
     type: FromCompositeFieldPath
EOF
```

## <a id="azure"></a>  Azure Flexible Server example `Composition`

The following `Composition` makes sure that all `FlexibleServers` are placed in the `westeurope` 
region and under the resource group `tap-psql-demo`.

> **Note** Setting the `FlexibleServerFirewallRule` to start at `0.0.0.0` and end at `255.255.255.255`
> allows access to the PostgreSQL Server from any IP address and is not recommended in a production
> environment.

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  labels:
    provider: azure
  name: xpostgresqlinstances.bindable.gcp.database.example.org
spec:
  compositeTypeRef:
    apiVersion: bindable.database.example.org/v1alpha1
    kind: XPostgreSQLInstance
  publishConnectionDetailsWithStoreConfigRef:
    name: default
  resources:
  - name: dbinstance
    base:
      apiVersion: dbforpostgresql.azure.jet.crossplane.io/v1alpha2
      kind: FlexibleServer
      spec:
        forProvider:
          administratorLogin: myPgAdmin
          administratorPasswordSecretRef:
            name: ""
            namespace: crossplane-system
            key: password
          location: westeurope
          skuName: GP_Standard_D2s_v3
          version: "12" #! 11,12 and 13 are supported
          resourceGroupName: tap-psql-demo
        writeConnectionSecretToRef:
          namespace: crossplane-system
    connectionDetails:
    - name: type
      value: postgresql
    - name: provider
      value: azure
    - name: database
      value: postgres
    - name: username
      fromFieldPath: spec.forProvider.administratorLogin
    - name: password
      fromConnectionSecretKey: "attribute.administrator_password"
    - name: host
      fromFieldPath: status.atProvider.fqdn
    - name: port
      type: FromValue
      value: "5432"
    patches:
    - fromFieldPath: metadata.uid
      toFieldPath: spec.writeConnectionSecretToRef.name
      transforms:
      - string:
          fmt: '%s-postgresql'
          type: Format
        type: string
      type: FromCompositeFieldPath
    - type: FromCompositeFieldPath
      fromFieldPath: metadata.name
      toFieldPath: spec.forProvider.administratorPasswordSecretRef.name
    - fromFieldPath: spec.parameters.storageGB
      toFieldPath: spec.forProvider.storageMb
      type: FromCompositeFieldPath
      transforms:
      - type: math
        math:
          multiply: 1024
  - name: dbfwrule
    base:
      apiVersion: dbforpostgresql.azure.jet.crossplane.io/v1alpha2
      kind: FlexibleServerFirewallRule
      spec:
        forProvider:
          serverIdSelector:
            matchControllerRef: true
          #! not recommended for production deployments!
          startIpAddress: 0.0.0.0
          endIpAddress: 255.255.255.255
  - name: password
    base:
      apiVersion: kubernetes.crossplane.io/v1alpha1
      kind: Object
      spec:
        forProvider:
          manifest:
            apiVersion: secretgen.k14s.io/v1alpha1
            kind: Password
            metadata:
              name: ""
              namespace: crossplane-system
            spec:
              length: 64
              secretTemplate:
                type: Opaque
                stringData:
                  password: $(value)
    patches:
    - type: FromCompositeFieldPath
      fromFieldPath: metadata.name
      toFieldPath: spec.forProvider.manifest.metadata.name
EOF
```

## <a id="gcp"></a> GCP Cloud SQL example `Composition`

The following `Composition` ensures that all CloudSQL PostgreSQL instances are placed in the 
`us-central1` region.

> **Note** The authorized network CIDR `0.0.0.0/0` allows access to the Cloud SQL from any IP address
> and is not recommended in a production environment.

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  labels:
    provider: gcp
  name: xpostgresqlinstances.bindable.gcp.database.example.org
spec:
  compositeTypeRef:
    apiVersion: bindable.database.example.org/v1alpha1
    kind: XPostgreSQLInstance
  publishConnectionDetailsWithStoreConfigRef:
    name: default
  resources:
  - base:
      apiVersion: database.gcp.crossplane.io/v1beta1
      kind: CloudSQLInstance
      spec:
        forProvider:
          databaseVersion: POSTGRES_14
          region: us-central1
          settings:
            dataDiskType: PD_SSD
            ipConfiguration:
              authorizedNetworks:
              - value: 0.0.0.0/0 # not recommended for production deployments!
              ipv4Enabled: true
            tier: db-custom-1-3840
        writeConnectionSecretToRef:
          namespace: crossplane-system
    connectionDetails:
    - name: type
      value: postgresql
    - name: provider
      value: gcp
    - name: database
      value: postgres
    - fromConnectionSecretKey: username
    - fromConnectionSecretKey: password
    - name: host
      fromConnectionSecretKey: endpoint
    - name: port
      type: FromValue
      value: "5432"
    name: cloudsqlinstance
    patches:
    - fromFieldPath: metadata.uid
      toFieldPath: spec.writeConnectionSecretToRef.name
      transforms:
      - string:
          fmt: '%s-postgresql'
          type: Format
        type: string
      type: FromCompositeFieldPath
    - fromFieldPath: spec.parameters.storageGB
      toFieldPath: spec.forProvider.settings.dataDiskSizeGb
      type: FromCompositeFieldPath
EOF
```
