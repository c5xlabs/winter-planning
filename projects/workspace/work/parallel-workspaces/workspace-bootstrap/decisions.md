# Workspace bootstrap decisions

## New-workspace ownership (decision:parallel-workspaces/workspace-bootstrap/creation-ownership)

**Decider:** product owner and engineering

**Open.** Choose seed/template configuration, source-control initialization, and handling of existing content during
creation; this blocks finalizing `product:create-unforked-workspace`, not the no-fork boundary, minimum common eager
safety hook, or refusal to overwrite an initialized workspace.

## Reconciliation scope (decision:parallel-workspaces/workspace-bootstrap/reconciliation-scope)

**Decider:** product owner and engineering

**Open.** Choose omitted-target scope, explicit inclusion of standalone and environment work, and how conflicts among
concurrent reconciliation invocations are reported; this blocks finalizing `product:reconcile-repositories` and
`product:bootstrap-recovery`, with environment mechanics owned by `epic:parallel-workspaces/environment-lifecycle`.

## Prune retention and approval (decision:parallel-workspaces/workspace-bootstrap/prune-retention)

**Decider:** product owner and engineering

**Open.** Choose how retained clean local-only history, stashes, obsolete generated copies, and uncertain ownership are
shown and whether an explicit discard path exists; this blocks finalizing `product:prune-managed-material`, not its
default preservation checks.
