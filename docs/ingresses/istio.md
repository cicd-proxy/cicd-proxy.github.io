# Istio Integration

The CI/CD Proxy works with Istio out-of-the-box.  It's been tested with Istio's side cars and can be deployed to use Istio as an `Ingress` controller.  When deploying with Istio, your helm chart will create the appropriate `Gateway`, `VirtualService`, and `Destination` objects depending on your configuration choices.  The `Service` objects will be named appropriately for Kiali to visualize the connections.

## Deployment

First, add a label to your CI/CD Proxy's namespace so Istio knows to add the envoy side cars:

```
kubectl label namespace cicd-proxy istio-injection=enabled
```

In your values.yaml, set:

1. `cicd_proxy.network.ingress_type: istio` - Tells the helm chart to create the correct objects
2. `cicd_proxy.network.istio.selectors` - List of labels that should be used to associate the `Gateway` with an istio controller.  Default is `istio: ingressgateway`

## No Load Balancer?

If you're running in a development environment or just don't have an external load balancer, you can configure Istio's `ingress-gateway` `Pod` to listen directly on 80 and 443 using this command:

```
kubectl patch deployments istio-ingressgateway -n istio-system -p '{"spec":{"template":{"spec":{"containers":[{"name":"istio-proxy","ports":[{"containerPort":15021,"protocol":"TCP"},{"containerPort":8080,"hostPort":80,"protocol":"TCP"},{"containerPort":8443,"hostPort":443,"protocol":"TCP"},{"containerPort":31400,"protocol":"TCP"},{"containerPort":15443,"protocol":"TCP"},{"containerPort":15090,"name":"http-envoy-prom","protocol":"TCP"}]}]}}}}'
```

[Return to deployment](/deployment/#pre-requisites)

## Known Issues & Troubleshooting

### Connection Refused or Reset

If your browser tells you that the connection is refused, it's one of:

1. Your load balancer isn't setup properly
2. If no load balancer, you haven't setup the host ports correctly
3. If the host ports are all configured properly and the load balancer is setup, did you create the `ou-tls-certificate` `Secret` in the `istio-system` `Namespace`?