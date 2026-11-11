---
epic: parallel-workspaces/coordinated-git
refinement: outline
status: planned
---

# Coordinate repositories without inventing one shared branch

Part of `initiative:parallel-workspaces`; product requirements live in [requirements.md](./requirements.md). Supports
parallel task delivery and safe history changes through explicit per-repository Git targets and outcomes.

## The problem

A task spans repositories with different main branches and upstreams. Bulk operations that borrow one sibling's target,
hide stale refs, or call every failure divergence can push or integrate the wrong work.

## The outcome

The engineer can inspect, connect, fetch, integrate, diff, and push a repository set while each worktree retains its own
tracking. Network and partial failures remain visible rather than obscured by a workspace summary.

## Depends on

- `epic:parallel-workspaces/environment-lifecycle`

## Left for later

Destructive pointer changes and untracked deletion belong to `epic:parallel-workspaces/safe-history`; stack rewrites to
`epic:parallel-workspaces/stacked-history`; standalone ref locks and updates to
`epic:parallel-workspaces/reproducible-inputs`. Dashboard presentation belongs to
`epic:agent-ready-operation/workspace-dashboard`.
