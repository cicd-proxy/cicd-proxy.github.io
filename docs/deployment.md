# Deploying the CI/CD Proxy

## Pre-requisites

Before deploying the CI/CD proxy you'll need three components in place:

***Ingress Controller -*** The CI/CD proxy is generally deployed with an Ingress controller.  The following controllers are supported directly by these helm charts:

| Ingress Controller | 
| ------------------ | 
| [NGINX](../ingresses/nginx) |
| [Istio](../ingresses/istio) |
| [Traefik](../ingresses/traefik) |

***Internal Certificate -*** Create a TLS `Secret` in the proxy's namespace called `kube-oidc-proxy-tls`.  The connection between the CI/CD proxy and the Ingress controller should be encrypted, since the CI/CD proxy will be receiving bearer tokens.  

***External Certificate-*** Create a TLS `Secret` in the proxy's namespace called `cicd-proxy-tls-certificate`.  This step can be skipped for Istio.

## Helm Chart Configuration

Once your Ingress and `Secret`s are in place, the next step is to configure your helm chart's values.yaml.  You'll need two items from your workflow engine:

***Issuer -*** The OIDC issuer tells the proxy how to validate the incoming token.  Each provider has a different provider,  The [Workflow Systems](/workflow_systems/) section contains specific configuration information for various workflow systems such as GitHub and GitLab.

***Workflow Identity-*** Each workflow has a unique identity, usually stored in the `sub` claim of a token.  Just as with the issuer, the [Workflow Systems](/workflow_systems/) section contains specific configuration information for various workflow systems such as GitHub and GitLab.

Finally, you can configure your chart.  The `cicd_proxy.networking` section should be configured per your Ingress controller.  The next section to configure is the `cicd_proxy.oidc` section.  The minimum configuration options are:

| Option | Description | Example |
| ------ | ----------- | ------- |
| `cicd_proxy.oidc.audience` | This is what the `aud` claim in all inbound tokens ***must*** be.  Usually the URL of your CI/CD proxy without a trailing "/" | https://myproxy.domain.io |
| `cicd_proxy.oidc.issuer` | The issuer from your workflow engine or identity system.  See the [Workflow Systems](/workflow_systems/) section contains specific configuration information for various workflow systems such as GitHub and GitLab. |
| 

The last configuration point is a list of "users" to authorize access for.  These are the `sub` claims from your workflow system's OIDC token.  These are the identities that will be impersonated to your Kubernetes cluster.  

## Deployment

With your values.yaml completed, the last step is to deploy your CI/CD Proxy:

```
helm repo add tremolo https://nexus.tremolo.io/repository/helm/
helm repo update
helm install cicd-proxy tremolo/cicd-proxy -n cicd-proxy -f /path/to/values.yaml
```

## Next Steps

With your proxy deployed, the next steps are to:

1. Define RBAC roles for your workflows
2. Integrate your CI/CD Proxy with your workflow system

For #1, defining your RBAC `Role` or `ClusterRole` for what you want your workflow to be able to do.  Then create a `RoleBinding` or `ClusterRoleBinding` using your `sub` from your workflow's identity JWT as the subject.  For instance, to create a `RoleBinding` to update a `Deployment` from a GitHub action, you would use something like:

```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: update-deployment
  namespace: myapp
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: update-deployment
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: repo:mlbiam/k8s-build-and-deploy-container:ref:refs/heads/main
```

This will look slightly different for each workflow system, so look at the [Workflow Systems](/workflow_systems/) for specific details.