# GitHub

Every GitHub action run gets an identity unique to a repository and branch.  To use this identity, you first need to get a JWT for the CI/CD proxy audience.  Tremolo Security built a [GitHub action](https://github.com/marketplace/actions/generate-oidc-jwt) to simplify this process:

```yaml
# Get a token for the aud cicd.tremolo.dev, must match cicd_prxy.oidc.audience in your values.yaml
- name: get oidc token
  uses: tremolosecurity/action-generate-oidc-jwt@v1.1
  with:
    audience: "cicd.tremolo.dev"
    environmentVariableName: "JWT"

# Use the token in a kubectl command
- name: patch deployment
  run: |
        kubectl config set-cluster kubernetes --server=https://cicd.tremolo.dev
        kubectl config set-context kubernetes --cluster=kubernetes --user=cicdproxy
        kubectl config set-credentials cicdproxy --token=$JWT
        kubectl config use-context kubernetes
        kubectl patch deployment run-service -n myapp -p "{\"spec\":{\"template\":{\"spec\":{\"containers\":[{\"name\":\"pause\",\"image\":\"ghcr.io/${{ env.REPO }}:${{ env.TAG }}\"}]}}}}"
```

## RBAC

The `sub` of the token provided to the CI/CD proxy will have the form:

`repo:ORG_USER_NAME/REPO_NAME/ref:refs/heads/BRANCH` 

Where:

* ***ORG_USER_NAME -*** The organization or username of the repository
* ***REPO_NAME -*** The name of of the GitHub repository
* ***BRANCH -*** The branch the action is running against

So for instance, the `mlbiam` GitHub user runs an action in the `k8s-build-and-deploy-container` repository on the `main` branch will have the `sub: repo:mlbiam/k8s-build-and-deploy-container:ref:refs/heads/main` claim in the JWT.  This means that the `User` subject in your RBAC bindings need to reference `repo:mlbiam/k8s-build-and-deploy-container:ref:refs/heads/main`.  For instance:

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

Allows our action to do whatever the `Role` `update-deployment` allows.