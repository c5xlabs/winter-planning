---
epic: parallel-workspaces/safe-history
refinement: outline
status: planned
---

# Change branch positions and remove files deliberately

Part of `initiative:parallel-workspaces`; product requirements live in [requirements.md](./requirements.md). Supports
task reuse and stack maintenance through explicit destructive scope and work-preservation checks.

## The problem

An engineer reusing an environment can accidentally move every repository instead of one, strand an unpushed env branch
behind detached HEAD, or delete files no reflog can restore. A generic force switch hides those different risks.

## The outcome

The engineer can deliberately adopt a branch, reset selected worktrees, or remove untracked content with clear scope,
preflight, approval, and partial-result information. Ordinary synchronization does not perform these actions implicitly.

## Depends on

- `epic:parallel-workspaces/coordinated-git`

## Left for later

Stack boundary derivation and history replay belong to `epic:parallel-workspaces/stacked-history`. Environment removal
belongs to `epic:parallel-workspaces/environment-lifecycle`; declared artifact cleanup belongs to
`epic:running-environments/resource-provisioning`, not Git clean.
