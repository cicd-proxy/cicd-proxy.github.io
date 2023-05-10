# Traefik

CI/CD Proxy's helm charts can generate the correct `Service` and `Ingress` objects for the [Traefik](https://doc.traefik.io/traefik/providers/kubernetes-ingress/) Ingress Controller.  Traefik requires that a downstream service's certificate be explicitly trusted, either by trusting the certificate or its signing CA.  This means that you'll need to either:

* **Trust the `kube-oidc-proxy-tls` `Secret` - ** You can configure your Traefik controller to explicitly trust the certificates generated by OpenUnison
* **Disable downstream TLS verification -** Add the `--serverstransport.insecureskipverify=true` parameter to the `command` in Traefik's `DaemonSet`/`Deployment`

Once you decide how you want Traefik to trust or verify the proxy's internal certificate, configure your values.yaml to use Traefik by setting `cicd_proxy.network.ingress_type` to `traefik`.  The helm charts will create all of the appropriate `Service` annotations and `Ingress` configurations for you.  The charts assume that you have an insecure entrypoint called `web` and a secure entrypoint called `websecure`.  You can configure these defaults by adding a `traefik` section to the `network` block in your values.yaml:

```yaml
netowork:
  traefik:
    secure: true
    entrypoints:
      plaintext: web
      tls: websecure
```

*** Using kubectl exec/cp/port-forward on Managed Clusters***

The kubectl exec/cp/port-forward commands all use the SPDY protocol which is not supported by Traefik.  The helm charts configure Traefik to use pass-through TLS to interact directly with the kube-oidc-proxy pods.  The kubectl configuration file includes the `kube-oidc-proxy-tls` certificate instead of the `Ingress` certificate.  This does not impact interaction with the portal.



[Return to deployment](/deployment/#pre-requisites)