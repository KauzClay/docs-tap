
# Prerequisites for installing Supply Chain Security Tools - Store

The following prerequisites are required to install Supply Chain Security Tools (SCST) - Store by
itself, instead of installing it as part of a
[Tanzu Application Platform profile](../about-package-profiles.hbs.md).

These steps ensure that you have a valid certificate after installing SCST - Store.

1. Install the SelfSigned Cluster Issuer by running:

   ```console
   # Install `tap-ingress-selfsigned` ClusterIssuer
   cat << EOF > /tmp/tap-ingress-selfsigned-cluster-issuer.yaml
   apiVersion: cert-manager.io/v1
   kind: ClusterIssuer
   metadata:
     name: tap-ingress-selfsigned
   spec:
     selfSigned: {}
   EOF

   kapp deploy -a issuer -f /tmp/tap-ingress-selfsigned-cluster-issuer.yaml -y
   ```

1. Install SCST - Store and update the certificate by running:

   ```console
   cat << EOF > /tmp/ingress-issuer-cert.yaml
   apiVersion: cert-manager.io/v1
   kind: Certificate
   metadata:
     name: ingress-cert
     namespace: metadata-store
   spec:
     isCA: true
     dnsNames:
       - metadata-store.${INGRESS_DOMAIN}
     issuerRef:
       kind: ClusterIssuer
       name: tap-ingress-selfsigned
       group: cert-manager.io
     secretName: ingress-cert
     commonName: metadata-store-ca
   EOF
   kubectl apply -f /tmp/ingress-issuer-cert.yaml
   # `ingress-cert` is deleted. It will be automatically created again with newly applied ClusterIssuer cert
   kubectl delete secret ingress-cert -n metadata-store
   ```