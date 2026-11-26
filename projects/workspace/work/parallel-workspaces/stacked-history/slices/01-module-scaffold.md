---
name: module-scaffold
depends-on:
  - epic:parallel-workspaces/coordinated-git
code:
  - winter-workspace/crates/core/src/termination.rs::GitOperation
  - winter-workspace/crates/core/src/termination.rs::CommandError
---

# The restack modules and errors

Lands, in one change, the shared edits `context/architecture/changes.md` requires first of work built as parallel
changes: the module files present and empty, and the errors the later slices return and render. These are the modules of
[restack-plan.md](../spec/restack-plan.md) and [restack-execution.md](../spec/restack-execution.md), Domain, with
`ChainError`, `RestackError`, and the `CommandError` and `GitOperation` variants restack-plan.md adds. Shaped by
`engineering:typed-outcomes`.

## Change

- `lib.rs` declares `restack`; `restack.rs` declares the `pub(crate)` submodule `execute`, an empty file under
  `restack/`, so the plan and the execution each add to their own file.
- `restack.rs` holds `ChainError` and `RestackError` as restack-plan.md, Domain, gives them; `CommandError` gains
  `Chain(ChainError)`; `lib.rs` re-exports `ChainError` and `RestackError` by name, with the types their variants carry.
  The entries of later slices are re-exported by those slices, as the Domain visibility paragraphs of restack-plan.md
  and restack-execution.md list.
- `GitOperation` gains `Rebase`, rendered "rebase".
- `context/architecture/structure.md` gains the rows of `restack` and `restack/execute` and what each may call;
  `changes.md` gains the rows its tables require for each. `restack` is a domain entry a command file calls, so
  `COMMAND_FILE_NAMES` is unchanged.
- Nothing constructs the new variants yet; no command changes.

## Acceptance

- Unit tests of the rendering: each `ChainError` variant reads as restack-plan.md, Domain, gives it, the element in
  single quotes as typed in `GlobElement` and `ScopedElement`; `Rebase` renders "rebase", and a `GitFailure::Repo`
  carrying it names that operation and the repository.
- A unit test: `Termination::Command` of `CommandError::Chain` has the error type "Command" and exit 1.
- `cargo dev gate slice` exits 0.

## Out of scope

- `RefTokenError`, which `RestackError::RefToken` carries, and `CommandError::RefToken` come from
  `epic:parallel-workspaces/coordinated-git` ([ref-tokens.md](../../coordinated-git/spec/ref-tokens.md)).
- The branch read's `TrackingRead` comes from `epic:parallel-workspaces/coordinated-git`
  ([push.md](../../coordinated-git/spec/push.md)); `plan-reads` uses it.
