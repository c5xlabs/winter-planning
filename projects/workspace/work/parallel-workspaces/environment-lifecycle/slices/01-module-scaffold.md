---
name: module-scaffold
code:
  - winter-workspace/crates/core/src/lib.rs
  - winter-workspace/crates/core/src/termination.rs::CommandError
  - winter-workspace/crates/core/src/termination.rs::SelectionError
  - winter-workspace/crates/core/src/termination.rs::RepoMessage
  - winter-workspace/crates/core/src/termination.rs::GitOperation
---

# The environment lifecycle modules and error variants

Lands, in one change, the shared edits `context/architecture/changes.md` requires first of work built as parallel
changes: the module files present and empty, and the error variants the later slices render. These are the modules of
[env-names.md](../spec/env-names.md), [env-selection.md](../spec/env-selection.md), [init.md](../spec/init.md),
[list.md](../spec/list.md), and [destroy.md](../spec/destroy.md), Domain, with the `EnvNameError` of env-names.md, the
`SelectionError::UnknownEnv` of env-selection.md, and the `RepoMessage` and `GitOperation` variants init.md and
destroy.md list. Shaped by `engineering:typed-outcomes`.

## Change

- `lib.rs` declares `env_name`, `env_selection`, `env_create`, `env_destroy`, and `env_list`; `env_create.rs` declares
  the `pub(crate)` submodule `worktree`, an empty file under `env_create/`, so each later slice adds to its own file.
- `termination.rs` holds `EnvNameError`, `BranchFault`, and `ReservedLocation` as env-names.md gives them, and
  `CommandError` gains `EnvName(#[from] EnvNameError)`.
- `SelectionError` gains `UnknownEnv { env }`.
- `RepoMessage` gains `SourceCheckoutMissing`, `NotAWorktreeOf`, `HeadDetached`, and `BadUpstream` of init.md, and
  `EnvDirectoryMissing`, `Dirty`, `InventoryUnreadable`, and `WorktreesKept` of destroy.md; `GitOperation` gains
  `BranchExists`, `WorktreeAdd`, `WorktreeList`, `CommonDir`, `Tracking`, `SetUpstream`, and `WorktreeRemove`.
- Nothing constructs the new variants yet; no command changes.

## Acceptance

- Unit tests of the rendering: each `EnvNameError` variant names the name and its fault, and `Reserved` names the
  location it would collide with; `UnknownEnv` reads "no environment named '<env>': none is registered and
  <root>/<env> is not a directory"; `SourceCheckoutMissing` ends "Run `winter ws init` first."; `Dirty` reads "refusing
  to destroy — dirty worktrees: <entries>. Re-run with --force to bypass."; `WorktreesKept` names the env directory and
  the worktrees that could not be removed.
- A unit test: `Termination::Command` of `CommandError::EnvName` and of `CommandError::Selection(UnknownEnv)` each have
  exit 1.
- `cargo dev gate slice` exits 0.

## Out of scope

- `StrictAbort`, `StrictPhase`, `ProvisionTeardown`, and `ProvisionCause` of destroy.md belong to the provision teardown
  and destroy hook phases, which `epic:running-environments/resource-provisioning` and
  `epic:agent-ready-operation/executable-extensions` own (destroy.md, System).
- `EnvIndexError` and `AllocateError` live beside the allocation they describe and land with `allocation-rule` and
  `registry-writes`.
