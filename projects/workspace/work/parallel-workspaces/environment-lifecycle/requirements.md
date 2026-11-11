# Environment lifecycle requirements

For `epic:parallel-workspaces/environment-lifecycle`.

## Name and select environments safely (product:environment-targets)

An engineer can identify the exact environment an operation addresses, including custom names and partially created
environments. Selection follows `decision:parallel-workspaces/target-selection`.

Acceptance:

- Valid names are distinguishable from paths, reserved workspace locations, and runtime scope names before mutation.
- Enumeration distinguishes registered, present, incomplete, missing, and unrecognized environment materializations.
- A literal unknown target receives an actionable result; patterns cannot silently broaden beyond their stated scope.
- Environments remain discoverable when one declared project is absent or no longer the first catalog entry.
- Inspection provides concrete paths without creating worktrees, allocating slots, or running lifecycle commands.

## Create and reconcile task worktrees (product:environment-worktrees)

The engineer can obtain one worktree per declared project in an environment without overwriting an existing task branch.

Acceptance:

- New worktrees use the intended env branch and each repository's configured starting branch; reused branches are named.
- Missing source checkouts receive guidance, rather than silently selecting an unrelated repository or remote default.
- Reconciliation adds missing worktrees and reapplies supported identity, excludes, and lightweight bootstrap work.
- Pinned projects track their own main; non-pinned new worktrees can inherit an unambiguous sibling upstream.
- No inference occurs when siblings disagree or provide no connected target; existing operator tracking is preserved.
- Detached or parked worktrees are reported rather than silently reattached during ordinary reconciliation.
- Structural success does not claim dependencies, resources, data, or services are ready.

## Keep environment allocations stable (product:stable-environment-allocation)

The engineer can run several environments without two tasks receiving the same workspace allocation. Allocation policy
is settled by `decision:parallel-workspaces/environment-lifecycle/allocation-policy`.

Acceptance:

- A live environment retains its assigned identity and port-band allocation across repeated invocations.
- Configured named slots and dynamically assigned slots cannot collide, including concurrent creation by two processes.
- Capacity exhaustion refuses creation clearly instead of reusing an occupied or reserved slot.
- Inspection distinguishes an allocated slot from a hypothetical suggestion and does not reserve suggestions.
- Workspace-shared scope remains separate from feature environments; invalid ranges and changed config are explained.
- Allocation release occurs only when teardown has made reuse safe, not merely because a removal was attempted.

## Recover allocation and discovery drift (product:environment-state-recovery)

The engineer can reconcile machine-local environment records with disk and Git without forgetting live allocations.

Acceptance:

- Missing state differs from corrupt or unreadable state; corruption does not silently reset allocations to empty.
- Stale assignments, unregistered worktrees, duplicate slots, and out-of-range assignments are individually identified.
- Repair explains effects on existing environments and running resources before changing their assigned locations.
- Recovery does not delete branches or directories simply because a record is missing.
- Cross-process conflicts and interruption leave either the previous usable assignment or an explicit recoverable state.

## Retire environments without losing work (product:destroy-environments)

The engineer can preview and retire selected environments while retaining work that would otherwise be deleted. Approval
follows `decision:parallel-workspaces/destructive-approval`.

Acceptance:

- Preview names worktrees, residual files, allocation release, and configured teardown effects without running them.
- Dirty, unreadable, or unsafe targets refuse before destructive teardown; explicit discard consent names loss.
- Surviving source-clone branches are distinguished from history, stashes, or files that would disappear with a target.
- Teardown runs while required worktrees still exist; failed teardown either retains the environment or follows an
  explicitly selected non-strict policy with an unsuccessful overall result.
- Failure to remove a worktree cannot silently become permission to delete its remaining directory or release its slot.
- Retiring one environment preserves sibling environments and workspace-shared resources outside its declared effects.
- Missing or partly removed targets remain repairable without falsely reporting completed teardown.

## Continue after incomplete lifecycle work (product:environment-recovery)

The engineer can tell what remains after creation, reconciliation, or teardown stops. Execution policy follows
`decision:parallel-workspaces/partial-execution`.

Acceptance:

- Results distinguish preflight refusal from partial execution and name changed, retained, failed, and unattempted work.
- Independent repository failures do not erase successful outcomes or silently skip later environments.
- Repeating an operation either resumes safely or identifies operator action needed; no global rollback is implied.
- Allocation records and managed ownership remain consistent with surviving work and pending cleanup.
