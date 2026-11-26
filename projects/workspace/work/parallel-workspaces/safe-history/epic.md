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
behind detached HEAD, or delete files no reflog can restore. When such a command stops part-way, the engineer cannot
tell which repositories already moved.

## The outcome

The engineer can deliberately adopt a branch, reset selected worktrees, or remove untracked content with clear scope,
preflight, approval, and partial-result information, and find the commit each moved pointer left behind. Ordinary
synchronization does not perform these actions implicitly.

## Depends on

- `epic:parallel-workspaces/repository-catalog`
- `epic:parallel-workspaces/environment-lifecycle`
- `epic:parallel-workspaces/coordinated-git`

## Left for later

Stack boundary derivation and history replay belong to `epic:parallel-workspaces/stacked-history`. Environment removal
belongs to `epic:parallel-workspaces/environment-lifecycle`; declared artifact cleanup belongs to
`epic:running-environments/resource-provisioning`, not Git clean.
