# Workspace bootstrap requirements

For `epic:parallel-workspaces/workspace-bootstrap`.

## Create a user-owned workspace (product:create-unforked-workspace)

An engineer can create a new workspace using an installed Workspace component rather than cloning Winter's framework
source. Initial content is settled by `decision:parallel-workspaces/workspace-bootstrap/creation-ownership`.

Acceptance:

- Creation establishes discoverable workspace configuration and a route to the selected component's guidance.
- The new workspace records its selected Workspace release through the agreed Platform pin contract under
  `decision:platform/launcher-shim/version-selection`, without changing another workspace's selection.
- Initial placement includes context-navigation's minimum eager safety text and common hook through `AGENTS.md` and a
  `CLAUDE.md` shim, without waiting for harness-specific projection or requiring command execution during context load.
- Creation refuses an already initialized workspace instead of silently becoming reconciliation or overwriting it.
- Existing files in an otherwise eligible target are preserved or explicitly identified as blocking creation.
- The resulting workspace needs neither embedded CLI source nor copied inherited framework documentation to operate.
- Source-control initialization and seed content are explicit, not permission to rewrite an existing repository.

## Reconcile declared repositories deliberately (product:reconcile-repositories)

An engineer can align selected source and standalone checkouts with the inventory, then repeat the operation after
configuration changes without resetting their branches or work.

Acceptance:

- Scope distinguishes source checkouts, standalones, and requested environment reconciliation; omission is explained.
- Missing clones are created; existing locations are verified rather than assumed to be the intended repository.
- Identity, declared excludes, and lightweight bootstrap commands can be reapplied with per-target outcomes.
- Existing branch positions and operator-owned tracking are not silently reset by ordinary reconciliation.
- Missing clone sources and failed bootstrap commands are actionable; unaffected targets remain reported.
- Dependency installation, resource provisioning, and service startup are not implicit structural reconciliation.

## Preserve workspace-owned and generated content (product:managed-content-ownership)

An engineer can reconcile generated support files and excludes without replacing their instructions or leaking ignore
rules into a sibling workspace checkout.

Acceptance:

- Managed content is distinguishable from user-owned text; updates preserve unrelated files, comments, and rules.
- Initial or repeated placement of the common safety hook preserves existing `AGENTS.md` and `CLAUDE.md` instructions;
  conflicting ownership is explained rather than resolved by replacing the user's file with a generated shim.
- Runtime artifacts and managed clone locations do not pollute the workspace repository's ordinary status.
- A workspace that is itself a linked worktree has independent managed excludes from its siblings and source checkout.
- Removing one managed target retracts only that target's ownership, including the last target in a group.
- Read-only inspection of ownership or excludes performs no Git configuration writes.

## Inspect and remove obsolete material safely (product:prune-managed-material)

The engineer can preview obsolete managed clones and generated entries, then remove only items whose deletion will not
discard work. Retention rules follow `decision:parallel-workspaces/workspace-bootstrap/prune-retention`.

Acceptance:

- Preview identifies candidate paths, ownership evidence, retained items, and reasons for refusal without deletion.
- Undeclared arbitrary user directories are not disposable merely because they sit beside managed repositories.
- Dirty or unreadable repositories, linked worktrees, local-only history, and stashes are protected unless a separately
  defined explicit discard operation authorizes the exact loss.
- Skipping confirmation does not bypass preservation checks or expand candidate scope.
- Failure reports removed, retained, and unattempted items; ownership records are not erased for retained data.

## Recover from interrupted reconciliation (product:bootstrap-recovery)

An engineer can understand and resume incomplete creation or reconciliation without guessing which repositories or
bootstrap commands ran.

Acceptance:

- Results distinguish completed, failed, and unattempted steps and identify surviving partial materializations.
- Repeating supported reconciliation does not duplicate repositories or erase work left after interruption.
- Concurrent supported bootstrap operations either converge without lost ownership or refuse conflicting effects.
- Arbitrary user bootstrap commands are identified as external effects whose safe repetition is the author's obligation.
