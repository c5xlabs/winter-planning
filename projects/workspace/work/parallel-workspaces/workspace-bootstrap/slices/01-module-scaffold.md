---
name: module-scaffold
code:
  - winter-workspace/crates/core/src/termination.rs::RepoMessage
  - winter-workspace/crates/core/src/termination.rs::GitOperation
  - winter-workspace/crates/core/src/termination.rs::CommandError
  - winter-workspace/crates/core/src/pins.rs::Ports
  - winter-workspace/crates/dev/src/architecture.rs::COMMAND_FILE_NAMES
---

# The bootstrap modules, errors, and shared ports type

Lands, in one change, the edits `context/architecture/changes.md` requires first of work built as parallel changes: the
module files present and empty, the error variants, the shared `Ports`, and the names no command file may hold. These
are the modules of [managed-content.md](../spec/managed-content.md), Domain,
[reconciliation.md](../spec/reconciliation.md), Domain, [creation.md](../spec/creation.md), Domain, and
[prune.md](../spec/prune.md), Domain, with the `RepoMessage` and `GitOperation` variants reconciliation.md lists and
`BootstrapError` of creation.md. Shaped by `engineering:typed-outcomes`.

## Change

- `lib.rs` declares `managed_block`, `workspace_exclude`, `agents_md`, `bootstrap`, and `prune`; `bootstrap.rs` declares
  the `pub(crate)` submodules `checkout` and `create`, each an empty file under `bootstrap/`, so each later slice adds
  to its own file.
- `Ports` moves from `pins.rs` to `crates/core/src/ports.rs`, re-exported from `pins.rs` and `lib.rs` under its present
  names; `materialize` and every other caller compile unchanged.
- `bootstrap.rs` holds `BootstrapError` with `AlreadyInitialized` and `JsonNeedsForce`; `CommandError` gains
  `Bootstrap(BootstrapError)`; `lib.rs` re-exports `BootstrapError`.
- `RepoMessage` gains `NoUrl`, `NotARepository`, `OriginMismatch`, `CommandFailed`, `CommandNotStarted`, and
  `UnsafeLocation`; `GitOperation` gains `Clone`, `Config`, `ExcludeLocate`, `Toplevel`, `OriginUrl`, `LocalOnly`, and
  `StashList`.
- `workspace_exclude` and `managed_block` join `COMMAND_FILE_NAMES` in `crates/dev/src/architecture.rs`, its pinned test
  renamed for the new count: a command file reaches them only through a domain entry.

## Acceptance

- Unit tests of the rendering: `NoUrl` reads "no `url` declared in config; cannot clone"; `CommandFailed` reads
  "`<command>` exited with code <returncode>"; `OriginMismatch` names the declared URL and the found one, or that none
  is set; `UnsafeLocation` names the path and the reason.
- A unit test: `Termination::Command` of `CommandError::Bootstrap` has the error type "Command" and exit 1.
- A unit test of the command-file scan in `crates/dev`: a command file that names `workspace_exclude` or `managed_block`
  is a violation naming the file, the line, and the token.
- The existing `pin_materialize` component tests pass unchanged against the moved `Ports`.
- `cargo dev gate slice` exits 0.
