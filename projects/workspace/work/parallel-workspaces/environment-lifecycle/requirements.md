# Environment lifecycle requirements

For `epic:parallel-workspaces/environment-lifecycle`.

## Name and select environments safely (product:environment-targets)

An engineer can identify the exact environment an operation addresses, and a name that would point at the workspace, its
source checkouts, or a path outside it is refused under
`decision:parallel-workspaces/environment-lifecycle/env-name-grammar`. Selection follows `product:target-patterns`, and
discovery follows `decision:parallel-workspaces/environment-lifecycle/discovery-authority`.

Acceptance:

- A name such as `.`, `..`, `projects`, `a/b`, or `.hidden` is refused before anything is created, changed, or deleted.
- `ws list` and glob targets cover the aliases and recorded environments present on disk; a `ws destroy` literal names
  an environment by its directory or its record.
- `ws init --all` reports a Git registration whose environment directory is gone rather than recreating it.
- Inspection provides concrete paths without creating worktrees, allocating slots, or running lifecycle commands.

## Create and reconcile task worktrees (product:environment-worktrees)

The engineer can obtain one worktree per declared project in an environment without overwriting an existing task branch.
A worktree moved off its environment branch is treated as
`decision:parallel-workspaces/environment-lifecycle/head-state` decides.

Acceptance:

- A new worktree starts a branch named for the environment from the repository's local main, without an upstream; an
  existing local branch of that name is reused.
- Missing source checkouts receive guidance, rather than silently selecting an unrelated repository or remote default.
- Reconciliation adds missing worktrees and reapplies supported identity, excludes, and lightweight bootstrap work.
- Pinned projects track their own main; non-pinned new worktrees can inherit an unambiguous sibling upstream.
- No inference occurs when siblings disagree or provide no connected target; existing operator tracking is preserved.
- Detached or parked worktrees are reported rather than silently reattached during ordinary reconciliation.
- Creating a worktree sets its push default as `product:parallel-workspaces/upstream-push-default` says.
- Structural success does not claim dependencies, resources, data, or services are ready.

## Keep environment allocations stable (product:stable-environment-allocation)

The engineer can run several environments without Workspace assigning two tasks the same workspace allocation.
Allocation policy is settled by `decision:parallel-workspaces/environment-lifecycle/allocation-policy`.

Acceptance:

- A live environment keeps its recorded index across repeated invocations; a configuration change that would move it to
  another index is reported before it is applied.
- A recorded index that collides with an alias's slot or the buffer slot is refused rather than shared, and two
  processes creating environments at once never record the same slot.
- An index two environments already record, as a hand edit or the earlier tool can leave, is reported by name in
  `ws list` rather than refused.
- A full band refuses creation clearly before any environment directory is created.
- Inspection distinguishes an allocated slot from a hypothetical suggestion and does not reserve suggestions.
- Workspace-shared scope remains separate from feature environments; a recorded index outside the band is reported.
- An environment's slot is released only once its worktrees are removed or its directory is already gone.

## Recover allocation and discovery drift (product:environment-state-recovery)

The engineer can reconcile machine-local environment records with disk and Git without forgetting live allocations.

Acceptance:

- Missing state differs from corrupt or unreadable state; corruption does not silently reset allocations to empty.
- A stale Git registration, a recorded index outside the band, and a recorded index on an alias or buffer slot are each
  reported by name.
- Recovery does not delete branches or directories simply because a record is missing.
- Cross-process conflicts and interruption leave either the previous usable assignment or an explicit recoverable state.

## Retire environments without losing work (product:destroy-environments)

The engineer can preview and retire selected environments while retaining work that would otherwise be deleted. Approval
follows `decision:parallel-workspaces/destructive-approval`, and teardown follows
`decision:parallel-workspaces/environment-lifecycle/teardown-policy`.

Acceptance:

- Preview names the worktrees and directory it would remove, and the leftovers it would keep, without running any
  teardown.
- A dirty worktree refuses the target before any teardown unless `--force` is given.
- Anything left in the environment directory once its worktrees are removed, such as files or clones Workspace did not
  create, is kept and listed as a warning, even with `--force`; the environment is still retired and its slot released.
- Removing a worktree keeps its environment branch in the source checkout.
- A failed worktree removal keeps the environment directory and its slot, and fails that target.
- Retiring one environment preserves sibling environments and workspace-shared resources outside its declared effects.
- Destroying an environment whose directory is already gone releases its slot and succeeds with a note.

## Continue after incomplete lifecycle work (product:environment-recovery)

The engineer can tell what remains after creation, reconciliation, or teardown stops. Execution policy follows
`decision:parallel-workspaces/partial-execution`.

Acceptance:

- Results distinguish preflight refusal from partial execution and name completed, failed, and unattempted targets.
- Independent repository failures do not erase successful outcomes or silently skip later environments.
- Repeating an operation either resumes safely or identifies operator action needed; no global rollback is implied.
