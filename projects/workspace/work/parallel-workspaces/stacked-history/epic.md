---
epic: parallel-workspaces/stacked-history
refinement: outline
status: planned
---

# Keep dependent task history after a lower branch changes

Part of `initiative:parallel-workspaces`; product requirements live in [requirements.md](./requirements.md). Supports
engineers maintaining dependent feature work across several project repositories.

## The problem

When a lower task is amended or squash-landed, rebasing its dependants against today's branch tips can replay the wrong
commits or drop the dependant's own work. A conflict halfway across repositories leaves the engineer to work out what
already moved before running anything again.

## The outcome

The engineer can declare a stack, inspect its replay boundaries, and move dependants onto updated predecessors while
preserving their own changes. A conflict stops with enough information to resolve it with Git and re-run the same
command to continue.

## Depends on

- `epic:parallel-workspaces/coordinated-git`
- `epic:parallel-workspaces/safe-history`

## What this epic is not

Restack never publishes rewritten history or chooses a hosting provider's review workflow; force-pushing a restacked
branch is done with Git itself.
