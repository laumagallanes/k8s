# CI/CD on Kubernetes with GitLab, Harbor, SonarQube, Rancher, Sentry and Argo CD

This repository documents a real-world approach for building a **self-hosted CI/CD platform on Kubernetes distributions such as K3s and RKE2**.

The original implementation was built for an on-prem environment and included several operational lessons learned during deployment, troubleshooting and day-to-day maintenance. This public version removes all client-specific information, credentials, domains, hostnames and internal network details.

## Goals

- provide a reproducible reference architecture for a self-hosted platform
- document practical deployment decisions, not just ideal diagrams
- show how the CI/CD flow connects source control, build, image registry, quality gates and GitOps delivery
- keep the material usable for teams running Kubernetes in constrained or partially manual environments

## Stack

- **K3s** and **RKE2** as Kubernetes distributions
- **GitLab** for source control and CI pipelines
- **GitLab Runner** for executing build and deploy jobs
- **Harbor** as private container registry
- **SonarQube** for static code analysis
- **Rancher** for cluster administration
- **Sentry** for error tracking and release visibility
- **Argo CD** for GitOps-style application delivery
- **Nginx / Ingress / TLS** for service exposure, proxying and certificates

## Repository structure

```text
README.md
k3s/
  00-overview.md
  02-k3s-deployment.md
  03-gitlab.md
  04-gitlab-runners.md
  05-recreate-gitlab.md
  06-sonarqube.md
  07-harbor.md
```

## Security note

All sensitive information from the original environment has been intentionally removed or generalized. If you adapt these notes, keep secrets, internal endpoints and credentials outside the repository.
