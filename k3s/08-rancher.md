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

## Small installation how-to with Helm

A practical Helm-based installation flow looks like this:

### 1. Prepare prerequisites

Before installing Rancher, make sure you already have:

- a working K3s cluster
- ingress available
- DNS or at least a planned hostname for Rancher
- TLS strategy defined, whether temporary or proper
- Helm installed on the administration machine

### 2. Add the Rancher Helm repository

```bash
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
helm repo update
```

### 3. Create the namespace

```bash
kubectl create namespace cattle-system
```

### 4. Install cert-manager if your setup requires it

In many Rancher deployments, certificates are part of the bootstrap path. The exact setup may vary, but cert-manager is a common dependency if you are not terminating everything elsewhere.

### 5. Install Rancher with Helm

A typical pattern is:

```bash
helm install rancher rancher-stable/rancher \
  --namespace cattle-system \
  --set hostname=rancher.example.internal \
  --set replicas=1
```

Adjust the hostname and replica count to fit the environment.

### 6. Wait for readiness

After installation, allow time for:

- certificates to be issued
- containers to start
- ingress routes to come up

Check status with:

```bash
kubectl get pods -n cattle-system
kubectl get ingress -n cattle-system
```

### 7. Retrieve the bootstrap password

If you did not predefine your own bootstrap password, get it with:

```bash
kubectl get secret --namespace cattle-system bootstrap-secret \
  -o go-template='{{.data.bootstrapPassword|base64decode}}{{ "\n" }}'
```

Then complete first login in the Rancher UI.

## Installation result seen in the notes

The recovered notes show a successful installation with Helm, ending in a deployed release and bootstrap-password generation.

That confirms the intended deployment model was:

- install Rancher in-cluster
- expose it through ingress
- wait for certificates and ingress to become ready
- complete first-login bootstrap from the generated password

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
