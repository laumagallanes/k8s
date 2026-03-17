# Sentry Deployment

Sentry adds runtime error tracking and release visibility to the platform.

## Objective

Deploy Sentry in the K3s cluster and make it available to applications so teams can track errors, exceptions and release health over time.

## Role in the platform

Sentry belongs to the operational side of the stack.

It complements CI/CD by answering a different question:

- CI tells you whether a build and deployment succeeded
- Sentry tells you what broke after the code reached users or test environments

## Prerequisites

Before deploying Sentry, make sure you have:

- a working K3s cluster
- ingress already available
- Helm installed
- a storage plan for persistent data
- a hostname and TLS strategy if Sentry will be exposed through ingress

## Create a namespace

```bash
kubectl create namespace sentry
```

## Install with Helm

Add the chart repository and refresh indexes:

```bash
helm repo add sentry https://sentry-kubernetes.github.io/charts
helm repo update
```

Export the default values so they can be customized:

```bash
helm show values sentry/sentry > sentry-values.yaml
```

## Customize values

The recovered notes point to a pattern with persistence and ingress enabled.

Example shape:

```yaml
persistence:
  enabled: true
  storageClass: "longhorn"
  size: 10Gi

ingress:
  enabled: true
  ingressClassName: "nginx"
  hosts:
    - host: sentry.example.internal
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: sentry-tls
      hosts:
        - sentry.example.internal
```

Adjust the storage class, hostname and TLS secret to match the environment.

## Install Sentry

```bash
helm install sentry sentry/sentry -n sentry -f sentry-values.yaml
```

## Optional ingress manifest

If ingress is not fully handled by the chart values, an explicit ingress can be created.

Example pattern:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: sentry-ingress
  namespace: sentry
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - sentry.example.internal
      secretName: sentry-tls
  rules:
    - host: sentry.example.internal
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: sentry-web
                port:
                  number: 9000
```

Apply it with:

```bash
kubectl apply -f sentry-ingress.yaml
```

## Validate the deployment

Check the pods:

```bash
kubectl get pods -n sentry
```

Then confirm that the UI is reachable through the chosen hostname.

## Create an admin user

If needed, an admin user can be created from the Sentry web container.

Example pattern:

```bash
sentry createuser --email user@example.com --superuser
```

This should be run in the appropriate Sentry web container or execution context.

## Release tracking with `sentry-cli`

The field notes also show how Sentry can be integrated into delivery pipelines through `sentry-cli`.

Install the CLI:

```bash
curl -sL https://sentry.io/get-cli/ | SENTRY_CLI_VERSION="2.21.2" sh
```

Then define environment variables such as:

```bash
export SENTRY_URL="https://sentry.example.internal/"
export SENTRY_AUTH_TOKEN="<TOKEN>"
export SENTRY_ORG="example-org"
export SENTRY_PROJECT="example-project"
export VERSION="${CI_COMMIT_SHA}"
```

And create a release:

```bash
sentry-cli releases new "$VERSION" \
  --org="$SENTRY_ORG" \
  --project="$SENTRY_PROJECT" \
  --auth-token "$SENTRY_AUTH_TOKEN"
```

## Why Sentry matters here

This platform is not just about building and deploying. It is also about shortening the loop between:

- code change
- deployment
- runtime feedback

That is where Sentry becomes useful, especially once the platform starts serving real applications.

## Common failure modes

Typical problems include:

- ingress not exposing the service correctly
- insufficient storage or resources
- CLI tokens not set correctly in CI
- wrong organization or project values in release automation
- TLS mismatch when applications or CLI try to reach Sentry
