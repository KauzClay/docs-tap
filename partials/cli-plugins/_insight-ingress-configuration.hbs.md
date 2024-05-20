<!-- Configure certificate and endpoint for insight cli when ingress is enabled -->

Set the endpoint host to:

```yaml
metadata-store.INGRESS-DOMAIN
```

Where `INGRESS-DOMAIN` is the value of the `ingress_domain` property in your deployment YAML

Example:

```yaml
metadata-store.example.domain.com
```

> **Note** In a multicluster setup, a DNS record is required for the domain. The following
> instructions for single cluster setup do not apply, skip to the [Set the target](#set-target)
> section.

## <a id="single-cluster"></a> Single-cluster setup

In a single-cluster setup, a DNS record is still recommended. However, if no accessible DNS record
exists for the domain, edit the `/etc/hosts` file to add a local record:

```console
ENVOY_IP=$(kubectl get svc envoy -n tanzu-system-ingress -o jsonpath="{.status.loadBalancer.ingress[0].ip}")

# Replace with your domain
METADATA_STORE_DOMAIN="metadata-store.example.domain.com"

# Delete any previously added entry
sudo sed -i '' "/$METADATA_STORE_DOMAIN/d" /etc/hosts

echo "$ENVOY_IP $METADATA_STORE_DOMAIN" | sudo tee -a /etc/hosts > /dev/null
```

## <a id="set-target"></a> Set the target

{{> 'partials/cli-plugins/insight-set-target' ingress=true }}