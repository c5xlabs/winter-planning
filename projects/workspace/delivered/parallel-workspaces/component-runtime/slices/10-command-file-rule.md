---
name: command-file-rule
depends-on:
  - result-writing
code:
  - winter-workspace/crates/core/src/commands/repo/remove.rs::remove_with
  - winter-workspace/crates/core/src/commands/ws/index.rs::answer
  - winter-workspace/crates/core/src/workspace_root.rs::CONFIG_FILE
  - winter-workspace/crates/core/src/config_raw.rs::LOCAL_CONFIG_FILE
  - winter-workspace/crates/core/src/state_registry.rs::STATE_FILE
---

# Command files reduced to arguments, one call, and the mapping of its outcome

Makes the two command files what [command-registration.md](../spec/command-registration.md), System, says a command file
holds, and adds the check that section lists, under `engineering:additive-registration` and
`engineering:domain-io-boundaries`. The removal transaction and the index lookup move into domain modules that take the
file port, and the workspace's file names get one owner. Behaviour does not change: the order of checks of
[repository-removal.md](../../../../work/parallel-workspaces/repository-catalog/spec/repository-removal.md), Interfaces,
and the reads of [index-lookup.md](../../../../work/parallel-workspaces/environment-lifecycle/spec/index-lookup.md),
Interfaces, are the ones the code has today.

## Change

- `crates/core/src/workspace_files.rs` is the one owner of the workspace's file names, each relative to a workspace
  root: `pub(crate) const CONFIG_FILE` (`.winter/config.toml`), `LOCAL_CONFIG_FILE` (`.winter/config.local.toml`), and
  `STATE_FILE` (`.winter/state.toml`). `workspace_root.rs`, `config_raw.rs`, and `state_registry.rs` import them and
  define none; no other module spells one of the three paths outside a test.
- `crates/core/src/repository_catalog.rs` holds the removal, moved from the command file with its tests. Its entry is
  the one call of `repo remove`:

  ```rust
  pub(crate) struct Removed { pub kind: RepositoryKind, pub name: String }    // the two parts of `target`

  #[derive(Debug, thiserror::Error)]
  pub(crate) enum RemoveError {
      #[error(transparent)] Catalog(#[from] CatalogError),
      #[error(transparent)] Config(#[from] ConfigError),
  }

  pub(crate) fn remove(
      files: &dyn ReadFiles,
      writer: &dyn WriteFiles,
      start: &Path,
      target: &str,
      local: bool,
  ) -> Result<Removed, RemoveError>;
  ```

  - `remove` performs the four checks in the spec's order: a `target` without `/` is `CatalogError::TargetShape` before
    any file is read; the root is found from `start`; a type that is neither word is `UnknownKind`; then the transaction
    runs on the root joined with `LOCAL_CONFIG_FILE` when `local` is set, else with `CONFIG_FILE`.
  - Private to the module: the `Removal` outcome enum, the `BlockEdit` type, the transaction and its form that takes the
    edit (validate as TOML, edit, verify with `config_edit::lacks_one_block`, replace through `WriteFiles::update`), and
    the parse of the type word.
  - `Removal::NotFound` and `Removal::Entangled` become the `CatalogError` of the same name; an invalid file and a
    failed update stay `ConfigError::FileRead` on the edited file's path.
- `crates/core/src/index_lookup.rs` holds the one call of `ws index`:

  ```rust
  pub(crate) fn answer(files: &dyn ReadFiles, start: &Path, name: &str) -> Result<IndexAnswer, ConfigError>;
  ```

  It finds the root, loads the two keys, reads the registry, and calls `env_index::lookup`. The rule that a slot count
  beyond `u32` is looked up as the largest `u32` moves here with its comment. `env_index.rs` stays pure.
- `crates/core/src/commands/repo/remove.rs` is left with `REGISTRATION`, `args`, `Params` and `params`, `document`,
  `line`, and `run`. `run` makes the one call
  `repository_catalog::remove(cx.files, cx.file_writer, cx.cwd, params.target, params.local)` and maps its outcome:
  - `Ok` to the result written through `output`;
  - `RemoveError::Catalog` to `Termination::Command(CommandError::Catalog(..))`;
  - `RemoveError::Config` to `Termination::Config(..)`.

  `document` and `line` take the `Removed` value; the type word they write is the `Display` of its kind, which is the
  word given.
- `crates/core/src/commands/ws/index.rs` is left with `REGISTRATION`, `args`, `Params` and `params`, `source_name`,
  `document`, `line`, and a `run` that makes the one call `index_lookup::answer(cx.files, cx.cwd, params.name)` and maps
  an error to `Termination::Config(..)`.
- The architecture rules gain the command-file rule of command-registration.md, System: a function of
  `crates/dev/src/architecture.rs` that answers, for a file under `crates/core/src/commands/` and its text, one
  violation per line and listed name, through a `Violation` variant that renders the file, the line, and the name; for a
  file elsewhere it answers nothing. `crates/dev/tests/architecture.rs` applies it to every library source, in the test
  that already walks `crates/core/src`.
  - What it checks: that no command file's text holds one of the names of the spec's list — the file port's types, the
    five modules a domain entry stands on, the two TOML crates, and the names of writing and of an internal failure.
  - What it cannot check: whether a file that holds none of them makes one call or carries logic of its own; a module
    that is not on the list; a name brought in by a glob import. The list is a deny-list, and the change that adds a
    module only a domain entry may call extends it.
- The contributor documents follow:
  - `context/architecture/changes.md`, A command, states what a command file holds and the names it may not hold, and
    says that the architecture test checks the names and what it cannot see. Its "What stays a shared edit" table gains
    the row for a module only a domain entry may call: that list in `crates/dev/src/architecture.rs`.
  - `context/architecture/structure.md`, Modules, gains rows for `workspace_files.rs`, `repository_catalog.rs`, and
    `index_lookup.rs`. The rows of `workspace_root.rs`, `config_raw.rs`, and `state_registry.rs` stay as they are.
- Files touched: `crates/core/src/{workspace_files,repository_catalog,index_lookup}.rs` (new),
  `crates/core/src/{lib,workspace_root,config_raw,state_registry}.rs`, `crates/core/src/commands/repo/remove.rs`,
  `crates/core/src/commands/ws/index.rs`, `crates/dev/src/architecture.rs` (the rule, its `Violation` variant and
  `Display` arm, its unit tests, and the module's opening comment), `crates/dev/tests/architecture.rs`,
  `context/architecture/{structure,changes}.md`.
- Unchanged: `config_edit.rs`, `env_index.rs`, `files.rs`, `termination.rs`, `parse.rs`, `output.rs`, the public
  re-exports of `lib.rs`, every file under `crates/core/tests/`, `scenarios/`, and `context/verification/matrix.md`,
  whose `workspace:architecture` row already names the library rules.

## Acceptance

- Unit tests of `repository_catalog.rs`:
  - the transaction tests that were in the command file, moved with their in-memory writer and unchanged in what they
    assert: the NaN, infinity, date, and nested-block cases; an edit that removes another block than the one it reports,
    and each edit that would change another entry, is refused with the file unchanged and never replaced; an edit that
    removes the one block replaces the file once;
  - only `project` and `standalone` are type words;
  - through `remove` with in-memory files: a target without `/` is `TargetShape` with no workspace above `start`; an
    unknown type word is `ConfigError::RootNotFound` with no workspace above and `UnknownKind` inside one; `local` edits
    the local file and leaves the shared one; a name no block has is `NotFound`; success returns the kind and the name
    of the target.
- Unit tests of `index_lookup.rs` with in-memory files: a recorded name answers from the registry; an unrecorded name
  answers its hash slot; an `envs_per_workspace` beyond `u32` answers as the largest `u32` does; no workspace above
  `start` is `ConfigError::RootNotFound`.
- Unit tests of the command-file rule:
  - a source at `crates/core/src/commands/repo/add.rs` that holds each name of the list in turn gives one violation
    naming that file, the line, and the name;
  - the same source at `crates/core/src/parse.rs` gives none;
  - a name inside a test module and a name inside a comment are both reported;
  - a source that names `crate::env_index`, `crate::output`, `crate::termination`, `crate::context`, and
    `crate::registry` gives none.
- `cargo test -p winter-workspace-dev` passes over the checkout: neither command file holds a listed name.
- `grep -rnE 'const (LOCAL_CONFIG_FILE|CONFIG_FILE|STATE_FILE)' crates/core/src` prints three lines, all in
  `workspace_files.rs`.
- `grep -cE '^(pub\(crate\) )?fn ' crates/core/src/commands/repo/remove.rs` prints 5 (`args`, `params`, `run`,
  `document`, `line`), and the same command on `crates/core/src/commands/ws/index.rs` prints 6 (`source_name` beside
  them).
- Every test file under `crates/core/tests/` passes with no edit: `git diff --stat` of the change lists none of them and
  nothing under `scenarios/`.
- `cargo dev gate full` exits 0, and the eight recorded scenarios pass.

## Out of scope

- A value that holds the located root and the loaded configuration of one invocation, reached through `Context`:
  [configuration-model.md](../../workspace-configuration/spec/configuration-model.md), Root discovery and Load;
  `epic:parallel-workspaces/workspace-configuration`.
- A write-failure variant of `ConfigError`, so that a failed update is no longer reported as `FileRead`: the same epic.
- `CatalogError::Entangled` and the return type of `remove_repository_block`, which repository-removal.md, Domain, does
  not list: `epic:parallel-workspaces/repository-catalog`.
- A broken required-argument invariant reported as a usage error under a hand-typed command path: the rest of this epic.
- A shared home for the in-process scaffold that each component test file defines: the rest of this epic.
- A file-system call spelled without the `std::fs` token, which the text scan of
  [repository-layout.md](../spec/repository-layout.md), Library rules, does not see in any library source: the rest of
  this epic.
