# Overview

This project documents a sanitized, real-world CI/CD platform built on **K3s** for an on-prem environment.

The goal is to describe how the moving parts fit together in practice:

- source control and CI with **GitLab**
- job execution with **GitLab Runner**
- image storage in **Harbor**
- static analysis with **SonarQube**
- cluster administration with **Rancher**
- runtime error tracking with **Sentry**
- GitOps delivery with **Argo CD**

## Main objective

Provide a repeatable reference for teams that need a self-hosted pipeline from commit to cluster, without depending entirely on managed cloud services.

## Context

This documentation comes from a real implementation, but everything environment-specific has been removed:

- no client names
- no internal domains
- no IP addresses
- no credentials or tokens
- no private application names

## Platform intent

The platform supports a flow where developers push code to GitLab, trigger build and validation stages, publish container images to Harbor, and deploy workloads to K3s using a GitOps approach.

## Why K3s

K3s is a practical fit for environments where:

- infrastructure is limited or heterogeneous
- operational simplicity matters
- teams want Kubernetes without the full weight of a larger distribution
- clusters may be assembled incrementally

## Core concerns addressed by this documentation

- how to deploy the platform components
- how to expose services internally and externally
- how to connect runners, registry and cluster
- how to handle TLS and internal certificate friction
- how to back up critical services
- how to debug common operational failures
