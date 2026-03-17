# Harbor Integration

Harbor is used as the private image registry for the CI/CD platform.

## Objective

Provide a registry that can receive images built by GitLab Runner and serve them back to workloads running on K3s.

## Role in the flow

A typical path looks like this:

1. developers push code to GitLab
2. GitLab Runner builds the image
3. the pipeline pushes the image to Harbor
4. K3s pulls the image from Harbor during deployment

That makes Harbor a critical bridge between CI and Kubernetes.

## Deployment notes

The original notes reference Harbor as an external service used by both the runner and the cluster. They also show that early deployment stages relied on a pragmatic setup before DNS and TLS were fully solved.

This is common in self-hosted environments: the registry works first, and elegance comes later.

## Common connectivity problem

One of the practical issues was that K3s needed to pull images from Harbor even when:

- internal DNS was not fully configured
- the registry was exposed over HTTP instead of HTTPS
- some nodes needed manual host resolution

## Temporary workaround with `/etc/hosts`

A transitional pattern is to define the registry hostname manually in `/etc/hosts` on every node.

Example shape:

```bash
sudo nano /etc/hosts
```

Then add an entry such as:

```text
<REGISTRY_IP> registry.internal
```

This is not the ideal end state, but it helps unblock the environment while internal DNS is still incomplete.

## Configure K3s to trust the registry endpoint

K3s can be configured through `/etc/rancher/k3s/registries.yaml`.

Example pattern:

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

This tells K3s where to find the registry and, if needed, how to authenticate.

## Apply the registry configuration

After editing the file, restart the appropriate K3s service.

On a server node:

```bash
sudo systemctl restart k3s
```

On a worker node:

```bash
sudo systemctl restart k3s-agent
```

## Validate image pulls

A simple way to verify the integration is to test an image pull directly.

Example pattern:

```bash
sudo crictl pull registry.internal/project/example:latest
```

If that succeeds, the node can reach Harbor and authenticate correctly.

## Validate with a test pod

You can also validate the full path by running a pod that pulls from Harbor:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: harbor-test
spec:
  containers:
    - name: harbor-test
      image: registry.internal/project/example:latest
```

Then apply it:

```bash
kubectl apply -f harbor-test.yaml
kubectl get pods
```

## Runner-side considerations

The field notes also suggest that CI had to account for an insecure registry during earlier stages. In practice, that means:

- the runner must resolve the Harbor hostname
- the build environment must trust the registry mode in use
- pipeline variables or Docker configuration may be needed for insecure registry behavior

If the runner can push but the cluster cannot pull, the problem is usually not Harbor itself but node-side trust or resolution.

## Why this should evolve

Using HTTP and host-file hacks is acceptable as a short-lived bootstrap strategy, but it should not be the final design.

A stronger end state is:

- proper internal DNS
- valid TLS certificates
- no manual host entries
- cleaner authentication handling

## Troubleshooting

If pulls fail:

- confirm hostname resolution on every node
- check `/etc/rancher/k3s/registries.yaml`
- restart K3s after changes
- inspect K3s logs

Example:

```bash
sudo journalctl -u k3s
```

On worker nodes, inspect the agent service if needed.

## Operational takeaway

Registry integration is one of the easiest places for a self-hosted platform to become fragile. Documenting the exact trust, DNS and pull path matters almost as much as documenting Harbor itself.
