# GitLab

GitLab acts as the source control and CI entry point for the platform.

## Role in the stack

GitLab is used for:

- hosting repositories
- running CI pipelines
- storing pipeline definitions
- coordinating build and deployment flow

## Deployment style

The original implementation notes point to a containerized self-hosted deployment with persistent volumes for:

- configuration
- logs
- application data

This is a good fit for small-to-medium on-prem setups, especially during early rollout.

## Practical concerns

### Persistence

Do not treat the GitLab container as disposable unless volumes are correctly backed up. The important state lives in persistent storage.

### Port mapping and reverse proxying

A recurring issue in the field notes is the mismatch between container ports, host ports and externally exposed endpoints.

Document clearly:

- internal container ports
- host port bindings
- public or internal reverse-proxy entrypoints
- TLS termination point

### HTTPS and certificates

If GitLab is published behind a reverse proxy or internal certificate chain, pipeline integrations and external tools may fail unless trust is handled consistently.

That affects:

- browser access
- runners
- registry access
- GitOps tools that need to read repositories

## Backup and recovery

The notes also indicate the need to recreate or restore GitLab using volume backups. A complete GitLab operations guide should cover:

- what volumes must be preserved
- how to stop the service safely
- how to export volume contents
- how to restore on another host
- how to validate the instance after recovery

## Troubleshooting themes seen in the notes

- disk usage and log growth
- service unavailability after storage pressure
- broken access behind reverse proxies
- environment mismatches between CI definitions and deployed infrastructure

## Recommendation

In this repository, GitLab should be documented not only as a CI tool, but as a stateful service with operational requirements. That includes storage, backups, TLS, and runner connectivity.
