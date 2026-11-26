---
name: worktree-reconciliation
depends-on:
  - worktree-git-operations
  - push-default
  - epic:parallel-workspaces/workspace-bootstrap
---

# One project's worktree in one environment

Adds `worktree::run` and `EnvWorktree` of [init.md](../spec/init.md), One worktree, steps 1 to 5: the location step that
creates or verifies the worktree, then identity, excludes, tracking, and `cmd`. Identity, excludes, and `cmd` are the
steps of [reconciliation.md](../../workspace-bootstrap/spec/reconciliation.md), One repository and Commands, which
`epic:parallel-workspaces/workspace-bootstrap` builds; the push default is `push-default`'s. Delivers
`product:environment-worktrees` for one worktree, `engineering:worktree-effect-confinement` at creation, and
`engineering:lifecycle-step-isolation`, under `decision:parallel-workspaces/environment-lifecycle/head-state` and
`decision:parallel-workspaces/environment-lifecycle/location-confinement`, both registered deviations.

## Change

- `crates/core/src/env_create/worktree.rs` gains `EnvWorktree` and `run`; `run` emits `InitEvent`s through its callback
  and answers whether no step reported a `RepoError`.
- Location: an absent location is created on the existing local `<env>` branch, else on a new one from the main branch,
  after the env directory is checked again for a link; a present one must pass the four checks of init.md or fail
  `NotAWorktreeOf`, and no later step runs.
- Tracking: a detached or parked worktree is reported and never reattached, and a parked one gets no tracking write; a
  pinned worktree is set to `origin/<main_branch>` and `push.default=upstream`; a non-pinned one takes the inferred
  upstream only when it was created in this run or has no tracking of its own. A tracking failure fails the project, and
  `cmd` still runs.
- `bootstrap.rs`'s `Action` gains `WorktreeCreated`, `UpstreamInferred`, `PinnedTrackingSet`, `WorktreeDetached`, and
  `WorktreeParked`, with their text.
- `EnvWorktree` is re-exported by name from `lib.rs`, and `run` as `reconcile_worktree`, since `process::run` holds
  `run` at the crate root, for its tests ([init.md](../spec/init.md), Domain); no command calls `run` yet.

## Acceptance

- Component tests with real git and a fixture source checkout: an absent location is created on a new `<env>` branch
  from the local main with no upstream and reported `worktree_created`; with a local `<env>` branch already present it
  is checked out and reported the same.
- Component tests: a registered worktree of the project is `exists`; a plain directory, a clone of the same remote, and
  a link to another environment's worktree at the location each fail `NotAWorktreeOf` and run no later step; an env
  directory replaced by a link before creation fails `NotAWorktreeOf` and creates nothing outside the workspace.
- Component tests: identity is written to the worktree, its excludes go to the common `info/exclude`, and each `cmd`
  runs in the worktree.
- Component tests, pinned: the first run sets `origin/<main>` and `push.default=upstream` and reports both in
  `pinned_tracking_set`; a second run writes nothing and emits no tracking event.
- Component tests, non-pinned: a created worktree with an inferred upstream gets it and `push.default=upstream`,
  reported `upstream_inferred`; an existing worktree tracking something else keeps it; no inferred upstream writes
  nothing.
- Component tests: a detached worktree reports `worktree_detached` and, pinned, fails tracking with `HeadDetached` while
  its `cmd` still runs; a worktree on another branch reports `worktree_parked` naming it, and its tracking configuration
  is unchanged.
- A unit test: a failure's text names the failed step and the steps not attempted, in init.md's order.
- `cargo dev gate slice` exits 0.

## Out of scope

- Step 6, the nested delegation, belongs to `epic:parallel-workspaces/nested-workspaces`
  ([delegation.md](../../nested-workspaces/spec/delegation.md)).
