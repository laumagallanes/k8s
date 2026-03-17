# Argo CD Deployment

Argo CD provides the GitOps delivery layer of the platform.

## Objective

Deploy Argo CD inside K3s and use it to synchronize application definitions from Git into the cluster.

## Role in the architecture

Argo CD belongs inside the cluster, alongside other cluster-native operational tools.

Its purpose is to bridge the gap between:

- what CI builds and publishes
- what Kubernetes should actually run

## Install Argo CD with Helm

A straightforward installation flow is:

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
kubectl create namespace argocd
helm install argocd argo/argo-cd -n argocd
```

## Verify installation

Check pods and services:

```bash
kubectl get pods -n argocd
kubectl get svc -n argocd
```

## Access options

Two practical access models are:

### Port-forward

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### Ingress through Traefik

Expose `argocd-server` through ingress if the platform already has a working ingress strategy.

## Retrieve the initial admin password

```bash
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d
```

The default username is:

```text
admin
```

## Connect a Git repository

Argo CD needs access to the Git repository that stores deployment definitions.

Example pattern:

```bash
argocd repo add https://github.com/example/repository.git \
  --username <USERNAME> \
  --password <PASSWORD>
```

In a production-quality setup, repository credentials should be handled more carefully than this example suggests.

## Create an application

A minimal application definition looks like this:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: backend-app
  namespace: argocd
spec:
  destination:
    namespace: default
    server: https://kubernetes.default.svc
  source:
    repoURL: https://github.com/example/repository.git
    targetRevision: main
    path: path/to/manifests
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Apply it with:

```bash
kubectl apply -f backend-app.yaml
```

## GitOps flow with CI

The intended delivery path is:

1. GitLab CI builds the image
2. the image is pushed to Harbor
3. the deployment repository is updated with the new image tag
4. Argo CD detects the change and syncs the application

That means Argo CD should not build artifacts. It should consume already-built deployment definitions.

## Example CI-side update pattern

A common GitOps-oriented pipeline step looks like this:

```yaml
deploy:
  stage: deploy
  script:
    - git clone https://$GIT_USER:$GIT_TOKEN@github.com/example/repository.git
    - cd repository
    - sed -i "s/tag:.*/tag: ${CI_COMMIT_SHA}/g" path/to/deployment.yaml
    - git commit -am "Update image tag to ${CI_COMMIT_SHA}"
    - git push origin main
```

## Common failure mode seen in the notes

One of the recovered errors shows Argo CD failing to create an application because the repository URL was malformed or inaccessible.

This is a classic problem.

Before blaming Argo CD, check:

- repository URL formatting
- protocol consistency
- extra spaces in the URL
- authentication method
- TLS behavior if the repo is self-hosted

## Validation checklist

After setup, verify:

- Argo CD UI is reachable
- repositories can be added successfully
- the application appears in Argo CD
- sync completes successfully
- workloads update when manifests change

## Common failure modes

Typical issues include:

- bad repository URL syntax
- repository not reachable from Argo CD
- TLS mismatch with self-hosted Git endpoints
- missing credentials
- image updated in Harbor but manifest not updated in Git
- application sync enabled, but source path wrong
