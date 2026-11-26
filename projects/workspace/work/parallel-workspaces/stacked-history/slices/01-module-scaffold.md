---
name: module-scaffold
code:
  - winter-workspace/crates/core/src/lib.rs
  - winter-workspace/crates/core/src/termination.rs::GitOperation
---

# The restack modules and error variant

Lands, in one change, the shared edits `context/architecture/changes.md` requires first of work built as parallel
changes: the module files present and empty, and the error variant the later slices render. These are the modules of
[restack-plan.md](../spec/restack-plan.md) and [restack-execution.md](../spec/restack-execution.md), Domain, with the
`GitOperation` variant restack-plan.md adds. Shaped by `engineering:typed-outcomes`.

## Change

- `lib.rs` declares `restack`; `restack.rs` declares the `pub(crate)` submodule `execute`, an empty file under
  `restack/`, so the plan and the execution each add to their own file.
- `GitOperation` gains `Rebase`, rendered "rebase".
- Nothing constructs the new variant yet; no command changes.

## Acceptance

- A unit test of the rendering: `Rebase` renders "rebase", and a `GitFailure::Repo` carrying it names that operation and
  the repository.
- `cargo dev gate slice` exits 0.

## Out of scope

- The branch read's `TrackingRead` comes from `epic:parallel-workspaces/coordinated-git`
  ([push.md](../../coordinated-git/spec/push.md)); `plan-reads` uses it.
