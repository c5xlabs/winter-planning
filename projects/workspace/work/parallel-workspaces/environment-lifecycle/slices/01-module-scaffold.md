---
name: module-scaffold
code:
  - winter-workspace/crates/core/src/termination.rs::CommandError
  - winter-workspace/crates/core/src/termination.rs::SelectionError
  - winter-workspace/crates/core/src/termination.rs::RepoMessage
  - winter-workspace/crates/core/src/termination.rs::GitOperation
  - winter-workspace/crates/dev/src/architecture.rs::COMMAND_FILE_NAMES
---

# The environment lifecycle modules and error variants

Lands, in one change, the shared edits `context/architecture/changes.md` requires first of work built as parallel
changes: the module files present and empty, the error variants the later slices render, the names no command file may
hold, and the visibility the entry tests need. These are the modules of [env-names.md](../spec/env-names.md),
[env-selection.md](../spec/env-selection.md), [init.md](../spec/init.md), [list.md](../spec/list.md), and
[destroy.md](../spec/destroy.md), Domain, with the `EnvNameError` of env-names.md, the `SelectionError::UnknownEnv` of
env-selection.md, the `EnvListError` of list.md, and the `RepoMessage` and `GitOperation` variants init.md and
destroy.md list. Shaped by `engineering:typed-outcomes`.

## Change

- `lib.rs` declares `env_name`, `env_selection`, `env_create`, `env_destroy`, and `env_list`; `env_create.rs` declares
  the `pub(crate)` submodule `worktree`, an empty file under `env_create/`, so each later slice adds to its own file;
  `env_list.rs` holds `EnvListError` alone, as list.md gives it.
- `termination.rs` holds `EnvNameError`, `BranchFault`, and `ReservedLocation` as env-names.md gives them, and
  `CommandError` gains `EnvName(#[from] EnvNameError)`.
- `SelectionError` gains `UnknownEnv { env, path }`, `path` being `<root>/<env>`.
- `RepoMessage` gains `SourceCheckoutMissing`, `NotAWorktreeOf`, `HeadDetached`, and `BadUpstream` of init.md, and
  `EnvDirectoryMissing`, `Dirty`, `InventoryUnreadable`, and `WorktreesKept` of destroy.md; `GitOperation` gains
  `BranchExists`, `WorktreeAdd`, `WorktreeList`, `CommonDir`, `Tracking`, `SetUpstream`, and `WorktreeRemove`.
- `lib.rs` re-exports by name `EnvNameError`, `BranchFault`, and `ReservedLocation` on the `termination` line, and
  `EnvListError` on a `pub use` line of its own; the entries of later slices are re-exported by those slices, as the
  visibility paragraph of each leaf's Domain lists.
- `env_selection` and `worktree::` join `COMMAND_FILE_NAMES` in `crates/dev/src/architecture.rs`, its pinned test
  renamed for the new count: only `env_destroy::select` and the target selection of
  `epic:parallel-workspaces/coordinated-git` call `reach`, and only `env_create` and `env_list` call `worktree`.
  `env_name`, `env_create`, `env_destroy`, and `env_list` are domain entries a command file calls and do not join it.
- `context/architecture/structure.md` gains the rows of the five modules, the `env_create/worktree` submodule, and what
  each may call; `changes.md` gains the rows its tables require for each.
- Nothing constructs the new variants yet; no command changes.

## Acceptance

- Unit tests of the rendering: each `EnvNameError` variant names the name and its fault, and `Reserved` names the
  location it would collide with; `UnknownEnv` reads "no environment named '<env>': none is registered and
  <path> is not a directory", naming the `path` it carries; `SourceCheckoutMissing` ends "Run `winter ws init` first.";
  `Dirty` reads "refusing to destroy — dirty worktrees: <entries>. Re-run with --force to bypass."; `WorktreesKept`
  names the env directory and the worktrees that could not be removed.
- A unit test: `Termination::Command` of `CommandError::EnvName` and of `CommandError::Selection(UnknownEnv)` each have
  exit 1.
- A unit test of the command-file scan in `crates/dev`: a command file that names `env_selection` or `worktree::` is a
  violation naming the file, the line, and the token.
- `cargo dev gate slice` exits 0.

## Out of scope

- `StrictAbort`, `StrictPhase`, `ProvisionTeardown`, and `ProvisionCause` of destroy.md belong to the provision teardown
  and destroy hook phases, which `epic:running-environments/resource-provisioning` and
  `epic:agent-ready-operation/executable-extensions` own (destroy.md, System).
- `EnvIndexError` and `AllocateError` live beside the allocation they describe and land with `allocation-rule` and
  `registry-writes`.
