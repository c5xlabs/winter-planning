---
name: index-command
depends-on:
  - epic:parallel-workspaces/component-runtime
---

# The first command: ws index, tested in process

Builds `ws index` as [index-lookup.md](../../../parallel-workspaces/environment-lifecycle/spec/index-lookup.md)
specifies it, to the extent the table "How far the proof implements an owner's contract" of
[method-proof.md](../spec/method-proof.md) gives, with unit and component tests and no recorded scenario yet. It is the
command half of the read-only proof of `engineering:method-proof`, under
`decision:rust-adoption/parity-validation/proof-behaviours`. The reference tool is not run in this change. Of the epic
it depends on, it needs command registration by file location and the workspace builder of the test harness; it uses
nothing an earlier change of this epic adds.

## Change

- `crates/core/src/commands/ws/index.rs` is the binary's first command file, in the shape of
  [command-registration.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-registration.md),
  Interfaces. The `ws` group appears in help because it now has a command.
- The file port arrives: `crates/core/src/files.rs` with `ReadFiles` and `OsFiles` as
  [repository-layout.md](../../../../delivered/parallel-workspaces/component-runtime/spec/repository-layout.md), What a
  command receives, gives them, without `WriteFiles`; `Context` gains `files`, and `run` binds `OsFiles` to it.
- The test module of `crates/core/src/parse.rs` supplies `files` where it builds a `Context`, and
  `context/architecture/structure.md` lists the new modules.
- `workspace_root.rs`, `config_raw.rs`, `state_registry.rs`, and `env_index.rs` exist, each implementing the sections of
  [configuration-model.md](../../../parallel-workspaces/workspace-configuration/spec/configuration-model.md) and
  [lock-and-state.md](../../../parallel-workspaces/workspace-configuration/spec/lock-and-state.md) that the method-proof
  System block names beside it. They take `&dyn ReadFiles` and never name `std::fs`.
- `env_index::lookup` is the `lookup` line of lock-and-state.md, Env index allocation, and returns an `IndexAnswer`, not
  a `Result`: registry first, else the alias position, else the hash slot. It has no probe loop, and `EnvIndexError` is
  not defined in this change: `allocate` and its band-exhaustion failure belong to
  `epic:parallel-workspaces/environment-lifecycle`.
- A recorded value is answered as recorded, with the integer type index-lookup.md, Domain, gives the registry.
- Errors: the `ConfigError` variants the foundation defines are returned as their sections say. An error one of those
  sections names that `termination.rs` does not hold yet is added there, spelled as the section spells it.
- The `--json` document is encoded with `serde_json`, as repository-layout.md, Cargo workspace, says.
- `crates/core` gains `sha1` and `serde_json`, and `winter-workspace-test-support` as a dev-dependency.
- Files touched: `crates/core/src/commands/ws/index.rs`,
  `crates/core/src/{files,workspace_root,config_raw,state_registry,env_index}.rs`,
  `crates/core/src/{lib,context,process,termination}.rs`, `crates/core/src/parse.rs` (its test module only, for the
  `Context` literal that gains the field), `crates/core/Cargo.toml`, `Cargo.lock`, `crates/core/tests/ws_index.rs`,
  `context/architecture/structure.md` (for the new modules).

## Acceptance

- Unit tests of `env_index::lookup` assert every row of lock-and-state.md, Fixed points, under the default aliases and
  slot count; that a recorded name returns its recorded value with `Registry`; that an alias returns `Suggested` with
  `alias` set; and that `feature-x`, with a registry that records `other` at 40, returns 40 with `Suggested`.
- Component tests in `crates/core/tests/ws_index.rs` call `run` in process with in-memory handles against a
  `WorkspaceBuilder` workspace:
  - a recorded name and an unrecorded name each give the JSON document of index-lookup.md, parsed from stdout, with
    `Termination::Success`, and a snapshot of every file under the workspace is unchanged;
  - an unrecorded name whose hash slot the registry records for another name gives that slot with `source` `suggested`;
  - a name the state file records at `-3`, and one it records at `5000000000`, each come back with that `index` and
    `source` `registry`;
  - a NAME holding a double quote and a backslash comes back equal in the parsed document's `name`;
  - without `--json`, stdout starts with the integer;
  - `env_aliases` in both configuration files: the local list replaces the shared one;
  - a working directory two levels below the workspace root finds it; a directory with no workspace above gives
    `Termination::Config` holding `ConfigError::RootNotFound`, and nothing on stdout;
  - a `.winter/config.toml` that is not valid TOML gives `ConfigError::FileRead` with the `Toml` cause;
  - `envs_per_workspace` smaller than the alias count allows gives `ConfigError::EnvsPerWorkspaceTooSmall`.
- `winter-workspace ws --help` lists `index`; `winter-workspace ws index` without NAME exits 2; `winter-workspace ws`
  alone exits 2 with the `ws` help on stderr and nothing on stdout.
- `crates/core/src/registry.rs` and `groups.rs` are unchanged, `parse.rs` is unchanged outside its test module, and no
  lint attribute was added or removed anywhere.
- `cargo dev gate full` exits 0.

## Out of scope

- Recording and replaying the command against the reference tool: `proof-read-only`.
- Allocation with probing, recording an index, and band exhaustion: `epic:parallel-workspaces/environment-lifecycle`.
- Every key, table, and fault the method-proof table leaves with `epic:parallel-workspaces/workspace-configuration`;
  every other `ws` command: `epic:parallel-workspaces/environment-lifecycle`.
