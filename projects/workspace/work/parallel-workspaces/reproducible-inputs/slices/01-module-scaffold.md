---
name: module-scaffold
depends-on:
  - epic:parallel-workspaces/repository-catalog
code:
  - winter-workspace/crates/core/src/termination.rs::ConfigError
  - winter-workspace/crates/core/src/files.rs::OsFiles
  - winter-workspace/crates/dev/src/architecture.rs::COMMAND_FILE_NAMES
---

# The shared edits the epic's parallel slices stand on

Lands, in one change, the edits that `context/architecture/changes.md` requires first of work built as parallel changes:
the module files present and empty, the error variants, the names no command file may hold, and the visibility the entry
tests need. These are the modules, `PinError`, and the `RepoMessage` causes of
[standalone-pins.md](../spec/standalone-pins.md), Domain, and of [content-identity.md](../spec/content-identity.md),
Domain, with `LockError` of
[lock-and-state.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/lock-and-state.md), Domain.
Shaped by `engineering:typed-outcomes`.

## Change

- `lib.rs` declares `git`, `config_lock`, `pins`, and `fingerprint`. `git.rs` and `fingerprint.rs` are empty;
  `config_lock.rs` holds `LockError` and `LockEntryFault` alone; `pins.rs` holds `PinError` and declares the
  `pub(crate)` submodules `outcome`, `resolve`, `materialize`, `advance`, `repin`, `freeze`, and `snapshot`, each an
  empty file under `pins/`, so each later slice re-exports from its own submodule on a `pub use` line of its own without
  editing `pins.rs`.
- `ConfigError` gains `Lock(LockError)`, rendered as the delivered lock errors are worded.
- `RepoMessage` gains `UnresolvableRef`, `DirtyRePin`, `NotPinned`, `NoPinnedNamed`, `NoStandaloneNamed`, and
  `MissingCheckout`, worded as the two specs quote them: a placeholder marked `!r` repr-quoted, any other as it is.
- `config_lock` and `git::` join `COMMAND_FILE_NAMES` in `crates/dev/src/architecture.rs`, its pinned test renamed for
  the new count. `pins` and `fingerprint` are domain entries a command file calls, and do not join it.
- Every name the visibility paragraph of standalone-pins.md, Domain, lists becomes `pub` inside its private module and
  is re-exported by name from `lib.rs`: the file port, the program port with `OsPrograms` and `OsPrograms::new`,
  `CommandTracer`, `Diagnostics` with `Log`, `Level`, and `Log::DEFAULT`, `LockError`, `LockEntryFault`, and `PinError`.
  Being public, none of what this slice adds is dead code, so no `allow` is needed for the gate's `-D warnings`.
- `context/standards/testing.md` gains the entry test: a component test that calls an entry re-exported from `lib.rs`
  against the git fixtures, for an entry no command of the library reaches yet or that another epic's command calls.
- `context/architecture/structure.md` gains the rows of the four modules, the `pins/` submodules, and what each may
  call; `changes.md` gains the rows its tables require for each.

## Acceptance

- Unit tests of the rendering: each new `RepoMessage` variant renders the spec's wording; in `NotPinned`, a name holding
  `'` renders in double quotes, as `repr` does; in `MissingCheckout` and the second name of `DirtyRePin` the name is
  unquoted.
- Unit tests of the `RepoError` built from each cause: `UnresolvableRef` and `MissingCheckout` carry the path as `cwd`
  and the other four carry none.
- A unit test: `ConfigError::Lock` of `UnsupportedVersion` renders `<path>: unsupported lock file version`.
- A unit test of the command-file scan in `crates/dev`: a command file that names `git::` or `config_lock` is a
  violation naming the file, the line, and the token.
- `cargo dev gate slice` exits 0.
