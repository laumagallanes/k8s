# GitLab Runners

GitLab Runner is the execution layer of the CI system.

## Role in the platform

Runners execute the jobs that:

- build frontend and backend artifacts
- build container images
- authenticate against registries
- publish images
- prepare deployment outputs for later sync

## Real-world constraint

The original notes show a very common on-prem problem: the runner may only work if internal connectivity is patched manually.

Typical examples include:

- host file entries for internal services
- incomplete DNS
- reverse proxies standing in front of registry services
- custom trust settings for internal certificates

This is not elegant, but it is real. The documentation should acknowledge it openly.

## Things to document cleanly

- where runners are installed
- whether they run on a dedicated VM or directly in the cluster
- which executor type is used
- how they reach GitLab
- how they reach Harbor
- how authentication is handled

## Pipeline responsibilities

From the recovered notes, the pipeline at least includes a build stage and artifact handling. In a fuller version, this section should cover:

- frontend build
- backend build
- docker build and push
- optional SonarQube analysis
- promotion flow between environments

## Common failure modes

- registry not reachable from runner
- TLS trust mismatch
- wrong host mapping
- no disk space on build node
- image builds succeed but deployment side cannot pull from registry

## Recommendation

Keep runner documentation strongly tied to network reality. In self-hosted systems, CI failures often come from infrastructure assumptions rather than pipeline syntax.
