<!-- Configure port forwarding to connect to the metadata store -->

Configure port-forwarding for the service so that the curl command can access SCST - Store.
You can configure port-forwarding in a separate terminal window or in the background.

From a separate terminal window, run:

```console
kubectl port-forward service/metadata-store-app 8443:8443 -n metadata-store
```

Alternatively, run the following command in the background:

```console
kubectl port-forward service/metadata-store-app 8443:8443 -n metadata-store &
```

### Edit your `/etc/hosts` file for port-forwarding

Use the following script to add a new local entry to `/etc/hosts`:

```console
METADATA_STORE_PORT=$(kubectl get service/metadata-store-app --namespace metadata-store -o jsonpath="{.spec.ports[0].port}")
METADATA_STORE_DOMAIN="metadata-store-app.metadata-store.svc.cluster.local"

# delete any previously added entry
sudo sed -i '' "/$METADATA_STORE_DOMAIN/d" /etc/hosts

echo "127.0.0.1 $METADATA_STORE_DOMAIN" | sudo tee -a /etc/hosts > /dev/null
```