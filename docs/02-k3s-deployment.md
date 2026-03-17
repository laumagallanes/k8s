# K3s Deployment

This section describes the K3s foundation required for the CI/CD platform.

## Intended role of the cluster

The cluster hosts platform services and application workloads, including components such as:

- Argo CD
- Rancher
- ingress / reverse proxy pieces
- target application deployments
- supporting observability or operations components

## Planning considerations

Before installation, define at least:

- node roles
- storage model
- ingress model
- certificate approach
- registry access strategy
- backup boundaries

## Typical node model

A practical small deployment usually separates responsibilities across nodes such as:

- server/control-plane nodes
- worker/application nodes
- an auxiliary node or VM for supporting services such as reverse proxy, DNS helpers or external-facing tooling

The exact sizing depends on workload and redundancy targets.

## Networking and service exposure

Based on the field notes, networking is one of the recurring pain points in this kind of setup. Document these decisions early:

- which services are exposed through ingress
- which services stay internal only
- whether Nginx or another reverse proxy fronts the cluster
- how TLS is terminated
- how internal DNS is resolved

## Registry connectivity

One of the practical issues in self-hosted environments is making cluster nodes and runners reach an internal Harbor registry consistently.

This can require:

- DNS records that resolve from both runners and cluster nodes
- or temporary insecure-registry / HTTP settings in controlled environments
- or host mappings while infrastructure is still incomplete

If insecure registry settings are used, they should be clearly marked as transitional.

## Ports and cluster communication

The original notes include a list of K3s-related ports used between nodes. In a production-quality document, these should be rewritten as a clean firewall matrix rather than copied raw.

Recommended final doc additions:

- control-plane communication ports
- kube-apiserver exposure
- ingress ports 80/443
- any load balancer VIPs or service IP assumptions

## Operational note

A surprising amount of deployment friction comes not from Kubernetes itself but from:

- missing DNS
- reverse proxy mismatches
- internal TLS trust
- storage pressure
- manually patched connectivity between runner, registry and cluster

That is why this repository treats networking and registry access as first-class documentation topics.
