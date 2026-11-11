---
epic: parallel-workspaces/workspace-bootstrap
refinement: outline
status: planned
---

# Create and reconcile a workspace without a framework fork

Part of `initiative:parallel-workspaces`; product requirements live in [requirements.md](./requirements.md). Supports
Workspace adoption through a user-owned definition and repeatable repository reconciliation.

## The problem

An engineer currently starts with a framework checkout and must preserve inherited tool source and documents while
customizing their workspace. Repairing drift or removing obsolete materializations can also threaten user-owned files.

## The outcome

The engineer creates a workspace definition in a new location, reconciles selected declared repositories, and safely
identifies and removes obsolete managed material without adopting a source fork or losing unrelated content.

## Depends on

- `epic:parallel-workspaces/repository-catalog`
- `epic:agent-ready-operation/context-navigation`

## Left for later

Environment creation belongs to `epic:parallel-workspaces/environment-lifecycle`; extension installation and harness
projection belong to `epic:agent-ready-operation/extension-management` and
`epic:agent-ready-operation/harness-projection`. Bootstrap places context-navigation's common eager safety hook in
`AGENTS.md` and the `CLAUDE.md` shim without waiting for those richer projections. Migration of existing framework forks
belongs to `epic:rust-adoption/workspace-migration`.
