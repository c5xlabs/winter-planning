# Environment lifecycle decisions

## Detached and parked worktrees (decision:parallel-workspaces/environment-lifecycle/head-state)

**Decider:** product owner

`ws init <env>` reports a worktree whose HEAD is detached, or on a branch other than `<env>`, and never checks `<env>`
out again; for a parked worktree it writes no tracking, because the branch the operator moved to is not the
environment's. This is a registered deviation: the reference reports neither and points a parked pinned worktree's
branch at `origin/<main>`, so a later push of that branch could land on main.

## Environment name grammar (decision:parallel-workspaces/environment-lifecycle/env-name-grammar)

**Decider:** product owner

A feature-environment name is one path component that `git check-ref-format --branch` accepts, is not `.` or `..`, does
not start with `.`, and is not `workspace`, `projects`, or the first component of any standalone location, compared
without regard to case or Unicode normalisation because macOS volumes ignore both; every env-level command refuses any
other name before a git call or write. This is a registered deviation: the reference refuses only `workspace` on
`ws init` and an empty name or `/` on `ws destroy`, so `ws destroy .` deletes the workspace and `ws init projects`
rewires the source checkouts.

## Location confinement (decision:parallel-workspaces/environment-lifecycle/location-confinement)

**Decider:** product owner

`ws init <env>` reports an existing worktree location as present, and `ws destroy` removes one, only when it is no link
and is a registered worktree of that project's own Git store, verified again just before the effect; `ws destroy` keeps
anything else as a leftover, also when the source checkout is gone, and refuses an env directory that resolves outside
the workspace root, so neither command acts on a checkout or directory it did not make for that environment. These are
registered deviations: the reference reports any existing path as present, deletes any directory at a worktree's
location when the source checkout is gone, and removes an env directory wherever a link leads.

## Teardown preservation and failure policy (decision:parallel-workspaces/environment-lifecycle/teardown-policy)

**Decider:** product owner and engineering

Teardown keeps the reference's order, except that a failed worktree removal keeps the directory and the index slot and
fails the target, and the env directory is removed only when it is empty once the declared worktrees are gone: anything
left, such as files, clones or worktrees config no longer declares, or other residue, is kept and listed as a warning,
whatever `--force` says, and so previewed, because Winter never deletes what it did not create. A target whose worktrees
were removed succeeds with leftovers kept, its slot released and its exclude block stripped, since the environment no
longer exists; the slot is also released when `ws destroy` finds the directory already missing (exit 0, with a note),
and a non-strict `on_env_destroy` failure fails the target (exit 1) without aborting teardown, while `--strict` stays as
the reference. These are registered deviations: the reference deletes the whole directory, also after a failed removal,
exits 0 on a hook failure, and leaves a missing environment's slot unreleasable.

## Environment identity and discovery (decision:parallel-workspaces/environment-lifecycle/discovery-authority)

**Decider:** product owner and engineering

Discovery keeps the reference's rules: `ws list` and glob targets take the aliases and registry entries confirmed on
disk, a `ws destroy` literal names a directory or a registry entry, and `ws init --all` finds environments from the
first declared project's worktree registrations. One registered deviation: a registration whose directory is gone is
reported and not recreated, where the reference silently resurrects the environment.

## Allocation policy and changes (decision:parallel-workspaces/environment-lifecycle/allocation-policy)

**Decider:** product owner and engineering

Allocation keeps the reference's rules: an alias takes its fixed slot first, then the registry's record, then a probe of
the registry's occupancy, with no host-port check. Registered deviations: an environment whose index would change from
its recorded one is reported before the change is applied, a recorded index that collides with an alias or the buffer
slot is refused, and a full band is detected before any directory is created; an index two names already record, as a
hand edit or the reference can leave, is reported as a finding and not refused, because refusing would block an
environment that already runs there.
