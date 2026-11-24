# Workspace bootstrap engineering requirements

For `epic:parallel-workspaces/workspace-bootstrap`.

## What already holds

- The standalone reconciliation of `ws init` applies each standalone's pin after its clone or verification through
  `materialize`, Materialize a pin in [standalone-pins.md](../reproducible-inputs/spec/standalone-pins.md), built by
  `epic:parallel-workspaces/reproducible-inputs`; this epic calls it and reports its result.

## Worktree-local Git configuration integrity (engineering:workspace-exclude-isolation)

Generated workspace excludes remain isolated when the workspace is a linked worktree, including a bare common Git
directory; enabling any required Git feature preserves existing layout settings and sibling operations. Serves
`product:managed-content-ownership`.

## Ownership-based deletion authority (engineering:prune-ownership-evidence)

Pruning requires positive managed-ownership evidence and revalidates candidate ancestry and preservation conditions at
deletion time. Unreadable evidence cannot authorize removal or ownership-record cleanup. Serves
`product:prune-managed-material`.

## Reconciliation effect isolation (engineering:bootstrap-effect-accounting)

Repository-local sequences retain order while independent effects use bounded concurrency; cross-process conflicts,
interruption, and external-command failures retain target attribution without claiming global rollback. Serves
`product:reconcile-repositories` and `product:bootstrap-recovery`.
