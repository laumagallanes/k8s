# Overview

This project documents a sanitized, real-world CI/CD platform built around **K3s** and supporting self-hosted tooling.

The idea is simple: keep the infrastructure understandable, reproducible and maintainable, even when the environment is on-prem and the setup grows in layers over time.

## High-level architecture

The platform is split into two main areas:

### 1. Devtools node

A dedicated **devtools** machine runs the external platform services that support the delivery flow.

Typical services hosted there include:

- **GitLab**
- **Harbor**
- **SonarQube**
- **GitLab Runner**

This keeps the CI and supporting tooling outside the Kubernetes cluster itself, which can simplify early deployments and make troubleshooting more direct.

### 2. K3s cluster

A **5-node K3s cluster** runs the Kubernetes-side platform components and application workloads.

Within the cluster, the platform may host components such as:

- **Rancher**
- **Argo CD**
- **Vault**
- ingress-related components
- application workloads
- supporting services added later

This separation makes it easier to reason about responsibilities:

- the devtools machine handles build-time and platform-adjacent services
- the cluster handles runtime orchestration and cluster-native tooling

## Typical flow

1. developers push code to GitLab
2. GitLab Runner builds the application
3. images are pushed to Harbor
4. deployment definitions are updated
5. Argo CD syncs changes into K3s
6. applications run in the cluster
7. Rancher helps operate the cluster
8. SonarQube and other tools provide quality feedback

## Why this layout is useful

This design is practical for teams that:

- are building their first serious self-hosted platform
- need clear ownership boundaries
- want Kubernetes without putting every single tool inside it from day one
- need a migration path from "just make it work" to "make it clean"

## What this repository focuses on

This repository is not meant to be a product brochure. It is a technical how-to based on field work.

The goal is to document:

- how the components were deployed
- how they were connected
- what assumptions matter
- where the fragile parts usually are
- how to recover from the inevitable disasters

## Sanitization rule

All customer-specific information, credentials, hostnames, IP addresses and internal references have been removed or generalized.
