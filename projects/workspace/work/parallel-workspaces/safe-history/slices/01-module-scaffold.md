---
name: module-scaffold
depends-on:
  - epic:parallel-workspaces/environment-lifecycle
  - epic:parallel-workspaces/coordinated-git
code:
  - winter-workspace/crates/core/src/termination.rs::CommandError
  - winter-workspace/crates/core/src/termination.rs::GitOperation
  - winter-workspace/crates/core/src/termination.rs::RepoMessage
---

# The history modules and error variants

Lands, in one change, the shared edits `context/architecture/changes.md` requires first of work built as parallel
changes: the module files present and empty, and the error variants the later slices render. These are the modules of
[safety-probes.md](../spec/safety-probes.md), [checkout.md](../spec/checkout.md), [reset.md](../spec/reset.md), and
[clean.md](../spec/clean.md), Domain, with the `HistoryError`, `CleanError`, and `RepoMessage` variants of clean.md, the
`CheckoutError` of checkout.md, the `ResetError` of reset.md, and the `GitOperation` variants safety-probes.md lists.
Shaped by `engineering:typed-outcomes`; `CheckoutError`, `ResetError`, and `CleanError` carry the `EnvNameError` of
`epic:parallel-workspaces/environment-lifecycle` and the `SelectionError` and `RefTokenError` variants of
`epic:parallel-workspaces/coordinated-git`, which land first.

## Change

- `lib.rs` declares `history`; `history.rs` declares the `pub(crate)` submodules `checkout`, `reset`, and `clean`, each
  an empty file under `history/`, so each later slice adds to its own file.
- `history.rs` holds `HistoryError` with `JsonNeedsForce`, and `CommandError` gains `History(#[from] HistoryError)`.
- `history/clean.rs` holds `CleanError`, `history/reset.rs` holds `ResetError`, and `history/checkout.rs` holds
  `CheckoutError`, as clean.md, reset.md, and checkout.md, Domain, give them; their entries return them, never a
  `Termination`.
- `RepoMessage` gains `NotAGitWorktree` and `UnrecognisedClean`, worded as clean.md gives them.
- `GitOperation` gains `AbandonmentProbe`, `UntrackedProbe`, `Reset`, and `Clean`, each rendered with the name
  safety-probes.md gives.
- `lib.rs` re-exports by name `HistoryError`, `CleanError`, and `ResetError`, and `CheckoutError` as
  `HistoryCheckoutError`, since `bootstrap::checkout::CheckoutError` holds that name at the crate root, with the types
  their variants carry; the entries of later slices are re-exported by those slices, as the visibility paragraphs of
  safety-probes.md and clean.md, Domain, list.
- `context/architecture/structure.md` gains the rows of `history` and its submodules `checkout`, `reset`, and `clean`,
  and what each may call; `changes.md` gains the rows its tables require for each. `history` is a domain entry a command
  file calls and does not join `COMMAND_FILE_NAMES`.
- Nothing constructs the new variants yet; no command changes.

## Acceptance

- Unit tests of the rendering: `AbandonmentProbe` reads "abandonment probe", `UntrackedProbe` "untracked probe", `Reset`
  "reset", and `Clean` "clean".
- A unit test: `Termination::Command` of `CommandError::History(JsonNeedsForce)` has exit 1.
- Unit tests of the rendering: `NotAGitWorktree` reads "<path> is not a git worktree"; `UnrecognisedClean` names the
  path and the line.
- `cargo dev gate slice` exits 0.

## Out of scope

- `GitOperation::Rebase` and the `git rebase` and `git merge-base` spans belong to `ws restack` of
  `epic:parallel-workspaces/stacked-history` and to integration of `epic:parallel-workspaces/coordinated-git`
  (safety-probes.md, Domain).
- `TrackingRead`, `SetUpstream`, and `Config`, which this epic's calls also name, land with
  `epic:parallel-workspaces/coordinated-git`, `epic:parallel-workspaces/environment-lifecycle`, and
  `epic:parallel-workspaces/workspace-bootstrap`.
