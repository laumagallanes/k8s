# GitLab Deployment

GitLab acts as the source control and CI entry point for the platform.

## Objective

Deploy a self-hosted GitLab instance with persistent storage, ready to host repositories and CI/CD pipelines for workloads that will later be deployed into K3s.

## Deployment model

The original implementation used a containerized GitLab deployment with Docker volumes for persistence. This is a practical approach for small-to-medium on-prem environments, especially during the first rollout.

The critical point is simple: **the container is disposable, the data is not**.

## Persistent volumes

A minimal deployment should persist at least these areas:

- configuration
- logs
- application data

The original deployment mapped them as Docker volumes:

- `gitlab-config`
- `gitlab-logs`
- `gitlab-data`

## Basic deployment

Example deployment pattern:

```bash
docker run -d \
  --name gitlab \
  --restart always \
  -p 80:80 \
  -v gitlab-config:/etc/gitlab:Z \
  -v gitlab-logs:/var/log/gitlab:Z \
  -v gitlab-data:/var/opt/gitlab:Z \
  gitlab/gitlab-ce:latest
```

## What this gives you

This pattern is enough to:

- bring up a self-hosted GitLab quickly
- keep GitLab state outside the container filesystem
- survive container recreation if the volumes are preserved

## Practical considerations

### Port mapping

Keep the host port mapping aligned with whatever reverse proxy or internal routing model you use. If GitLab is published behind Nginx or another proxy, document both:

- the host port bound by Docker
- the external entrypoint used by users and integrations

### Persistence strategy

The volumes must be included in backup procedures. If they are lost, the GitLab instance is effectively gone even if the container image still exists.

### Version control of the platform itself

Avoid relying on `latest` forever in production. It is acceptable while prototyping, but a stable environment should pin versions and document upgrade strategy.

### TLS and reverse proxying

If GitLab is accessed behind an internal reverse proxy, TLS and host resolution must stay consistent across:

- browsers
- GitLab Runner
- Argo CD or other integrations
- registry clients if GitLab registry is ever used

## Operational checks

After deployment, verify at least:

- the container is healthy
- the web interface loads
- repositories can be cloned and pushed
- the CI pipeline can reach the instance
- persistent data survives a container restart

## Common failure modes

Typical issues in this type of deployment include:

- bad reverse proxy routing
- broken hostname resolution
- TLS mismatch
- insufficient disk space
- logs growing until the host runs out of storage

A GitLab deployment guide should always be paired with a restore guide, because GitLab is not just another stateless service.
