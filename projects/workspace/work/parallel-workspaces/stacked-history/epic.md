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
commits or drop the dependant's own work. A conflict halfway across repositories makes a blind rerun dangerous.

## The outcome

The engineer can declare a stack, inspect its replay boundaries, and move dependants onto updated predecessors while
preserving their own changes. Conflicts stop with enough information to resolve and resume deliberately.

## Depends on

- `epic:parallel-workspaces/safe-history`

## Left for later

Remote force-push policy remains publication behavior in `epic:parallel-workspaces/coordinated-git`; this epic does not
automatically publish rewritten history or choose a hosting provider's review workflow.
