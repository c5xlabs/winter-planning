---
name: module-scaffold
code:
  - winter-workspace/crates/core/src/lib.rs
  - winter-workspace/crates/core/src/termination.rs::CommandError
  - winter-workspace/crates/core/src/termination.rs::GitOperation
---

# The history modules and error variants

Lands, in one change, the shared edits `context/architecture/changes.md` requires first of work built as parallel
changes: the module files present and empty, and the error variants the later slices render. These are the modules of
[safety-probes.md](../spec/safety-probes.md), [checkout.md](../spec/checkout.md), [reset.md](../spec/reset.md), and
[clean.md](../spec/clean.md), Domain, with the `HistoryError` of clean.md and the `GitOperation` variants
safety-probes.md lists. Shaped by `engineering:typed-outcomes`.

## Change

- `lib.rs` declares `history`; `history.rs` declares the `pub(crate)` submodules `checkout`, `reset`, and `clean`, each
  an empty file under `history/`, so each later slice adds to its own file.
- `history.rs` holds `HistoryError` with `JsonNeedsForce`, and `CommandError` gains `History(#[from] HistoryError)`.
- `GitOperation` gains `AbandonmentProbe`, `UntrackedProbe`, `Reset`, and `Clean`, each rendered with the name
  safety-probes.md gives.
- Nothing constructs the new variants yet; no command changes.

## Acceptance

- Unit tests of the rendering: `AbandonmentProbe` reads "abandonment probe", `UntrackedProbe` "untracked probe", `Reset`
  "reset", and `Clean` "clean".
- A unit test: `Termination::Command` of `CommandError::History(JsonNeedsForce)` has exit 1.
- `cargo dev gate slice` exits 0.

## Out of scope

- `GitOperation::Rebase` and the `git rebase` and `git merge-base` spans belong to `ws restack` of
  `epic:parallel-workspaces/stacked-history` and to integration of `epic:parallel-workspaces/coordinated-git`
  (safety-probes.md, Domain).
- `TrackingRead`, `SetUpstream`, and `Config`, which this epic's calls also name, land with
  `epic:parallel-workspaces/coordinated-git`, `epic:parallel-workspaces/environment-lifecycle`, and
  `epic:parallel-workspaces/workspace-bootstrap`.
