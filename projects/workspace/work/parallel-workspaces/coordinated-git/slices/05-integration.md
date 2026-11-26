---
name: integration
depends-on:
  - module-scaffold
code:
  - winter-workspace/crates/core/src/pins/advance.rs::Integration
  - winter-workspace/crates/core/src/pins/advance.rs::IntegrateCheckout
---

# Integrating one checkout onto a target ref

Adds `integration::integrate` of [integration.md](../spec/integration.md): the fast-forward, the `--merge`, `--rebase`,
or `--no-ff` fallback, the autostash restore check, the lock, in-progress, and not-started classification, and the
abort, with no fetch and no merge or rebase left in progress. It also widens `Integration` of
[standalone-pins.md](../../../../delivered/parallel-workspaces/reproducible-inputs/spec/standalone-pins.md), Domain, by
the four new outcomes, which `advance` passes through. Delivers the integration bullets of
`product:integrate-selected-repositories`, `engineering:integration-recovery-evidence`, and the lock rule of
`engineering:shared-git-effects`, under `decision:parallel-workspaces/coordinated-git/integration-conflicts`, whose
split of what the reference calls diverged is a registered deviation.

## Change

- `crates/core/src/integration.rs` holds `Fallback`, `Integrated`, `AutostashStep`, and `integrate`; every non-zero exit
  but a lock is an outcome, and a lock is `GitFailure::Repo { Integrate, repo }` with Git's cause.
- With `autostash`, every merge and rebase call carries `--autostash`, and a call that exits 0 is checked for unmerged
  paths or a new stash entry.
- `pins::advance::Integration` gains `MissingRef`, `DirtyBlocked`, `AutostashFailed`, and `AbortFailed`, and its counts
  widen to `u64`, as `Integrated`'s are; `advance` passes each through like `Diverged`.
- `Fallback`, `Integrated`, `AutostashStep`, and `integrate` are re-exported by name from `lib.rs` for its tests
  ([integration.md](../spec/integration.md), Domain); no command calls them yet.

## Acceptance

- Component tests with real git: a behind checkout fast-forwards with its commit count; a level one is `UpToDate`; a
  diverged one is `Diverged` with its counts under `Fallback::None`, `Merged` under `Merge`, `Rebased` under `Rebase`;
  `NoFf` with nothing to integrate runs nothing and is `UpToDate`.
- Component tests: a target naming no commit is `MissingRef` and nothing else runs; a dirty tree blocking the
  fast-forward is `DirtyBlocked`; a conflicting merge and a conflicting rebase are each aborted, leaving no `MERGE_HEAD`
  or rebase directory, and answer `Diverged`.
- Component tests: with `--autostash`, a re-applied stash that conflicts after git exits 0 is `AutostashFailed` with
  `Restore`, and the stash entry stays; a refused save is `AutostashFailed` with `Save`.
- A component test: an abort that fails, forced by removing write access to the git directory after the conflict, is
  `AbortFailed` carrying the abort's error.
- A component test: with `index.lock` held, a fast-forward ends `GitFailure::Repo` with `Integrate` and Git's cause,
  never `Diverged`.
- The existing `pin_advance` component tests pass with the widened `Integration`.
- `cargo dev gate slice` exits 0.
