# K3s Deployment

This guide documents a practical **K3s deployment flow** for a self-hosted CI/CD platform.

It is based on a real implementation, but all environment-specific data has been removed or generalized.

## Objective

Deploy a K3s cluster capable of hosting the core services of the platform and the target application workloads.

This includes preparing the nodes, bootstrapping the cluster, joining workers, validating connectivity, and leaving the groundwork ready for ingress, registry access and higher-level tooling.

## Suggested topology

A simple working model for this kind of setup is:

- **control-plane nodes**
  - run the K3s server components
  - keep cluster state and coordination
- **worker nodes**
  - run application workloads and supporting services
- **supporting infrastructure node**
  - may host external tools such as GitLab, Harbor, SonarQube or reverse proxies, depending on the environment design

The exact number of nodes and sizing will vary by workload, but the important part is to define responsibilities early.

## Prerequisites

Before installing K3s:

- install a supported Linux distribution such as Ubuntu Server
- update packages
- configure hostnames
- assign static IP addresses if needed
- ensure SSH access with keys
- synchronize time with NTP
- confirm connectivity between all cluster nodes

Example base preparation:

```bash
sudo apt update && sudo apt upgrade -y
```

## Core components around the cluster

The surrounding stack typically includes:

- **Helm** for packaging and deploying applications
- **Traefik** as ingress controller
- **GitLab CI/CD** for building and publishing workloads
- **Harbor** as private registry
- **Longhorn** for persistent storage

## Install the first K3s server

Bootstrap the first server node:

```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --cluster-init" sh -
```

This initializes the cluster and makes the first control-plane node the initial server.

## Join additional server nodes

If the environment requires more than one control-plane node, add the remaining servers by pointing them to the first one.

Example pattern:

```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --server https://<SERVER_IP>:6443" sh -
```

In a production-ready version of this guide, token handling and secure join flow should be documented explicitly.

## Join worker nodes

On each worker node:

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://<SERVER_IP>:6443 K3S_TOKEN=<TOKEN> sh -
```

This connects the worker to the cluster and enables it to run workloads.

## Validate the cluster

Once the nodes are joined, confirm cluster health:

```bash
kubectl get nodes
```

Expected result:
- all intended server nodes appear
- all intended worker nodes appear
- nodes are in `Ready` state

## Persistent storage

For workloads that need persistence, a distributed storage layer such as **Longhorn** can be added.

Example installation:

```bash
helm repo add longhorn https://charts.longhorn.io
helm install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace
```

This prepares the cluster for stateful workloads.

## Reverse proxy and ingress

A practical pattern in self-hosted environments is to place **Nginx** in front of the cluster and forward traffic to **Traefik**.

Nginx can live on:
- a dedicated tools node
- or a control-plane node, if resource pressure is acceptable

The cleaner option is usually a separate tools or edge node.

Example Nginx pattern:

```nginx
server {
    listen 80;
    server_name example.internal;

    location / {
        proxy_pass http://TRAEFIK_IP:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

For HTTPS, add the TLS listener and certificates at the proxy layer or terminate TLS directly in the cluster, depending on the design.

## Exposing Traefik

If needed, Traefik can be exposed through a `LoadBalancer`-style service or through an external reverse proxy model.

Example shape:

```yaml
kind: Service
metadata:
  name: traefik
  namespace: kube-system
spec:
  type: LoadBalancer
  ports:
    - name: web
      port: 80
      targetPort: web
      protocol: TCP
    - name: websecure
      port: 443
      targetPort: websecure
      protocol: TCP
  selector:
    app.kubernetes.io/name: traefik
```

The final implementation will depend on whether the environment uses:
- MetalLB
- a dedicated load balancer VM
- a reverse proxy host
- static networking assumptions

## Registry access from K3s

One of the real-world problems in small self-hosted environments is making K3s pull images from Harbor before DNS and TLS are fully solved.

A transitional workaround is:

1. define local name resolution in `/etc/hosts`
2. configure `/etc/rancher/k3s/registries.yaml`
3. restart K3s services
4. test pulls using `crictl`

Example `registries.yaml` pattern:

```yaml
mirrors:
  "registry.internal":
    endpoint:
      - "http://registry.internal"
configs:
  "registry.internal":
    auth:
      username: "<USERNAME>"
      password: "<PASSWORD>"
```

Then restart:

```bash
sudo systemctl restart k3s
# or on worker nodes
sudo systemctl restart k3s-agent
```

And validate:

```bash
sudo crictl pull registry.internal/project/example:latest
```

### Important note

Using HTTP and local host mappings is acceptable as a temporary workaround in lab or constrained environments, but it should be replaced by:

- proper internal DNS
- valid TLS
- cleaner registry trust configuration

## Port considerations

The original field notes include a set of ports used for cluster communication. In the final version, these should be rewritten into a clean table with:

- control-plane ports
- API server exposure
- ingress ports
- storage-related communication if needed
- monitoring or management ports where relevant

That is more useful than dumping a raw list.

## Verification checklist

After installation, verify at least:

- `kubectl get nodes`
- ingress accessibility
- DNS or host resolution from runners and nodes
- successful image pulls from Harbor
- successful scheduling of a test pod
- persistent volumes provisioning correctly if Longhorn is installed

## Common issues

Typical problems in this kind of rollout:

- incomplete DNS
- reverse proxy misrouting
- TLS mismatch
- registry reachable from one node but not others
- workers missing host mappings
- disk pressure on a node
- image pull failures from Harbor
