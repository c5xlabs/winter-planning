---
initiative: running-environments
refinement: outline
epics:
  - scope-environment
  - resource-provisioning
  - service-orchestration
  - provider-tmux
  - provider-docker
  - provider-process-compose
---

# Run isolated environments

Engineers need a feature environment's resources and running services to agree on its identity without disturbing its
neighbors.

## The problem

A set of worktrees is not a working system. Credentials, ports, data, and processes must be prepared together, and an
uncertain shutdown or recovery can reach shared infrastructure or another task's resources. Replacing only the core CLI
would also leave Python required by the maintained first-party service providers.

## The outcome

`epic:running-environments/scope-environment` computes and explains runtime values using the authoritative environment
identity. `epic:running-environments/resource-provisioning` makes declared dependency, resource, data, and cleanup
actions repeatable, including the composed request for a working environment.
`epic:running-environments/service-orchestration` gives people precise selection, truthful readiness and status, and
attributable logs through one provider-independent interface.

`epic:running-environments/provider-tmux`, `epic:running-environments/provider-docker`, and
`epic:running-environments/provider-process-compose` each provide a native first-party runtime choice independently of
the other two. Their common obligations include isolation, explicit capability limits, fair log following, and cleanup
that proves ownership rather than guessing from a similar name.

## Left for later

`initiative:parallel-workspaces` owns structural creation, registry identity, and nested-workspace delegation.
`epic:agent-ready-operation/executable-extensions` supplies the external process boundary; provider actions stay with
service orchestration. `initiative:rust-adoption` owns declaration conversion, replacement evidence, and delivery.

## Stance

Workspace orchestrates existing runtimes rather than becoming a service supervisor. First-party behavior requires no
Python; arbitrary user programs and external providers keep their chosen languages and prerequisites. Built-in and
external providers share Winter-owned semantics and conformance obligations. Preparation failure cannot be disguised as
a ready workload, and preview never implies that trusted operator commands are sandboxed or reversible.
