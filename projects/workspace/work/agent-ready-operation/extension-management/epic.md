---
epic: agent-ready-operation/extension-management
refinement: outline
status: planned
---

# Choose what an installed extension contributes

Part of `initiative:agent-ready-operation`; product requirements live in [requirements.md](./requirements.md). Supports
context delivery, executable contributions, agent configuration, and harness projection.

## The problem

An engineer can clone a repository for its data without intending to adopt its hooks or agents. Today repository names,
manifest names, prefixes, and adoption modes give different answers about what is active, leaving stale contributions
when the configuration changes.

## The outcome

The engineer can inspect and reconcile a declared extension set, distinguish acquisition from activation, and see which
contributions are enabled and why. Disabling an extension does not delete its repository or silently execute cleanup.

## Depends on

- `epic:parallel-workspaces/workspace-configuration`
- `epic:parallel-workspaces/repository-catalog`
- `epic:parallel-workspaces/reproducible-inputs`
- `epic:agent-ready-operation/executable-extensions`

## Left for later

`epic:parallel-workspaces/repository-catalog` owns acquiring and locating repositories;
`epic:parallel-workspaces/reproducible-inputs` owns selected refs and locks.
`epic:agent-ready-operation/context-navigation` owns context sources and addressing.
`epic:agent-ready-operation/dependency-graph` owns graph inspection and `epic:agent-ready-operation/context-quality`
owns reference/dependency lint rules.

## What this epic is not

- A package registry, transitive dependency installer, dependency version solver, or replacement git workflow.
- Permission to execute an installed repository's code merely because its manifest was discovered.
