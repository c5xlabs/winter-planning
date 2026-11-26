---
name: module-scaffold
code:
  - winter-workspace/crates/core/src/termination.rs::Termination
  - winter-workspace/crates/core/src/termination.rs::EnvVarFault
---

# The scope environment modules and error vocabulary

Lands, in one change, the shared edits `context/architecture/changes.md` requires first of work built as parallel
changes: the `scope_env` module and its two submodule files, the error types every later slice returns or renders, and
the `Termination` variant `winter env` ends with. These are the Domain blocks of
[scope-identity.md](../spec/scope-identity.md) (`IndexFault`, `EnvError`), [env-command.md](../spec/env-command.md)
(`EnvCommandError`), [resolution.md](../spec/resolution.md) (`ResolutionError` and the types it carries, and
`EnvVarFault::InvalidKey` of Key validation at load), and [value-lookups.md](../spec/value-lookups.md) (`LookupRequest`,
`LookupError`, `LookupFailure`, `OutputFault`), with the first line and details of value-lookups.md, Failure rendering.
`LookupRequest` is the interface between resolution and a lookup, so `resolve-command-entries` and `run-lookup` build
against it in parallel. Shaped by `engineering:typed-outcomes` and `product:private-runtime-values`.

## Change

- `lib.rs` declares `scope_env`; `scope_env.rs` declares the `pub(crate)` submodules `resolve` and `lookup`, each a file
  under `scope_env/`, so each later slice adds to its own file.
- `scope_env.rs` holds `IndexFault`; `scope_env/resolve.rs` holds `ResolutionError`, `Unresolved`, `UnresolvedCause`,
  and `Unclaimed`; `scope_env/lookup.rs` holds `LookupRequest`, `LookupError`, `LookupFailure`, and `OutputFault`.
- `termination.rs` holds `EnvError` with its four variants, `Interrupted` included, and `Termination` gains
  `Env(Box<EnvError>)`, exit 1, as
  [results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
  Domain, gives it; `Termination::error_type` answers `Env`. An interrupted lookup ends as the
  `Termination::Aborted { interrupted }` that `epic:parallel-workspaces/workspace-bootstrap` adds; this epic adds no
  `Termination::Interrupted`, which is the service paths' alone.
- `scope_env.rs` holds `EnvCommandError` of env-command.md, the failure of the `winter env` domain entry.
- `EnvVarFault` gains `InvalidKey`.
- `lib.rs` re-exports by name the types this slice adds, each module's names on a `pub use` line of their own:
  `IndexFault`, `EnvError`, `EnvCommandError`, `ResolutionError`, `Unresolved`, `UnresolvedCause`, `Unclaimed`,
  `LookupRequest`, `LookupError`, `LookupFailure`, and `OutputFault`. The entries of later slices are re-exported by
  those slices, as the visibility paragraphs of scope-identity.md, resolution.md, value-lookups.md, and
  [consumers.md](../spec/consumers.md), Domain, list them.
- `context/architecture/structure.md` gains the rows of `scope_env`, `scope_env/resolve`, and `scope_env/lookup`, and
  what each may call; `changes.md` gains the rows its tables require for each. `scope_env` is the domain entry
  `commands/env.rs` calls and does not join `COMMAND_FILE_NAMES`.
- Nothing constructs the new types yet; no command changes.

## Acceptance

- Unit tests of the rendering: each `LookupFailure` but `Interrupted`, which is never rendered, and each `OutputFault`
  renders the first line of Failure rendering for its variant, `<band> key '<key>': command`<command>`<what>`, followed
  by the cwd, exit, and stderr details it has; `UnknownScope` reads "no env by that name is registered" with the scope;
  `RecordedIndex` names the scope, the index, and the fault; `InvalidKey` names the band and the key.
- A unit test per `ResolutionError` and `UnresolvedCause` variant: the text names the band and the key, and a token
  renders as declared.
- A unit test: `Termination::Env` has exit 1 and its `error_type` answer is `Env`.
- `cargo dev gate slice` exits 0.

## Out of scope

- Which code returns each error: `scope-identity`, `resolve-literals`, `resolve-command-entries`, `run-lookup`, and
  `env-key-validation`.
