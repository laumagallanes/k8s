# Rancher Deployment

Rancher is used as the cluster administration layer for the K3s environment.

## Objective

Deploy Rancher inside the cluster so operators can manage workloads, namespaces, projects and cluster resources through a central interface.

## Role in the platform

Rancher sits on the Kubernetes side of the architecture, not on the external devtools node.

That means its job is to help operate the cluster where the workloads actually run.

## Namespace

The original installation notes indicate Rancher was deployed in:

- `cattle-system`

That is the standard Rancher namespace and should be kept unless there is a strong reason to diverge.

## Installation result

The recovered notes show a successful installation with Helm, ending in a deployed release and bootstrap-password generation.

That tells us the intended deployment model was:

- install Rancher in-cluster
- expose it through ingress
- wait for certificates and ingress to become ready
- complete first-login bootstrap from the generated password

## General installation flow

A practical Rancher deployment guide should look like this:

1. ensure ingress is working
2. ensure certificates are available or planned
3. create or confirm the target namespace
4. install Rancher with Helm
5. wait for pods and ingress readiness
6. retrieve bootstrap credentials
7. complete initial setup in the web UI

## Bootstrap password retrieval

A standard way to retrieve the generated bootstrap password is:

```bash
kubectl get secret --namespace cattle-system bootstrap-secret \
  -o go-template='{{.data.bootstrapPassword|base64decode}}{{ "\n" }}'
```

This is useful if you did not define your own bootstrap password during installation.

## Operational note

Rancher may take a while to become ready after installation. This is normal while:

- certificates are issued
- pods settle
- ingress routes become active

Do not assume failure just because the UI does not appear instantly.

## Dependencies around Rancher

The field notes suggest Rancher was part of a broader cluster stack that also included or planned:

- Traefik with cert-manager
- Longhorn
- Vault
- additional auth integration
- GitOps tooling such as Argo CD or Fleet
- monitoring components

That is important because Rancher rarely lives alone. It tends to become part of the operational center of the cluster.

## Validation checklist

After installing Rancher, verify:

- Rancher pods are healthy in `cattle-system`
- ingress resolves correctly
- TLS behaves as expected
- the bootstrap password works
- the dashboard loads normally

## Common failure modes

Typical issues include:

- ingress not routing correctly
- certificates not being issued or trusted
- DNS not resolving the Rancher endpoint
- bootstrap confusion during first login
- partial readiness while services are still starting
