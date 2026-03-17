# CI/CD on K3s with GitLab, Harbor, SonarQube, Rancher, Sentry and Argo CD

This repository documents a real-world approach for building a **self-hosted CI/CD platform on K3s**.

The original implementation was built for an on-prem environment and included several operational lessons learned during deployment, troubleshooting and day-to-day maintenance. This public version removes all client-specific information, credentials, domains, hostnames and internal network details.

## Goals

- provide a reproducible reference architecture for a small-to-medium self-hosted platform
- document practical deployment decisions, not just ideal diagrams
- show how the CI/CD flow connects source control, build, image registry, quality gates and GitOps delivery
- keep the material usable for teams running Kubernetes in constrained or partially manual environments

## Stack

- **K3s** as lightweight Kubernetes distribution
- **GitLab** for source control and CI pipelines
- **GitLab Runner** for executing build and deploy jobs
- **Harbor** as private container registry
- **SonarQube** for static code analysis
- **Rancher** for cluster administration
- **Sentry** for error tracking and release visibility
- **Argo CD** for GitOps-style application delivery
- **Nginx / Ingress / TLS** for service exposure, proxying and certificates

## High-level flow

1. Developers push code to GitLab.
2. GitLab CI runs build, test and analysis stages.
3. Images are built and pushed to Harbor.
4. Deployment manifests or application definitions are updated.
5. Argo CD syncs the target environment from Git.
6. Applications run on K3s.
7. SonarQube and Sentry provide quality and runtime feedback.
8. Rancher helps operate and observe the cluster.

## What this repo is and is not

### This repo is
- a practical documentation project
- a sanitized knowledge base derived from a real implementation
- a starting point for building a similar platform

### This repo is not
- a dump of raw customer notes
- a secret store
- a turnkey product with every value hardcoded

## Planned documentation structure

```text
README.md
docs/
  00-overview.md
  01-architecture.md
  02-k3s-deployment.md
  03-gitlab.md
  04-gitlab-runners.md
  05-harbor.md
  06-rancher.md
  07-sonarqube.md
  08-sentry.md
  09-argocd.md
  10-networking-and-tls.md
  11-backups-and-operations.md
  12-known-issues.md
```

## Initial topics to document

### K3s deployment
- node roles
- cluster prerequisites
- networking basics
- ingress exposure model
- internal registry access

### GitLab and runners
- self-hosted GitLab deployment
- runner placement and connectivity
- build pipeline patterns
- artifact and image workflows

### Harbor integration
- image publication flow
- registry trust model
- dealing with internal DNS or insecure registry constraints in lab/on-prem setups

### GitOps delivery with Argo CD
- repository connectivity
- sync model
- common TLS and certificate pitfalls

### Operations
- backups
- storage and disk pressure
- troubleshooting failed deploys
- promotion between environments

## Security note

All sensitive information from the original environment has been intentionally removed or generalized. If you adapt these notes, keep secrets, internal endpoints and credentials outside the repository.

## Status

Work in progress. The first pass focuses on turning field notes into public, reusable documentation without leaking environment-specific data.
