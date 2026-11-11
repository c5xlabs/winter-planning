---
epic: running-environments/service-orchestration
refinement: outline
status: planned
---

# Operate services through one truthful interface

Part of `initiative:running-environments`; product requirements live in [requirements.md](./requirements.md). Supports
independently deliverable first-party providers, external providers, and required-service provisioning checks.

## The problem

Engineers cannot trust a common command if selectors widen during dispatch, missing status looks ready, or stopping log
following kills the application. Each backend currently exposes different gaps behind the same command names.

## The outcome

The engineer selects services consistently across providers, understands partial starts and shutdowns, and obtains
attributable status and fair live logs without learning a second backend-specific public control interface.

## Depends on

- `epic:running-environments/scope-environment`
- `epic:agent-ready-operation/executable-extensions`

## Left for later

Each native provider is delivered by its own epic in this initiative. Required-service checks integrate at
`product:running-environments/resource-provisioning/required-service-preconditions`; registry lifecycle integration
remains with `epic:parallel-workspaces/environment-lifecycle` and does not make this epic depend on provisioning.

## What this epic is not

- A Winter service daemon or a generic passthrough for arbitrary backend flags.
- A requirement that every built-in provider be a separate executable.
