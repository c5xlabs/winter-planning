---
name: layered-load
depends-on:
  - epic:parallel-workspaces/component-runtime
code:
  - winter-workspace/crates/core/src/config.rs::load
  - winter-workspace/crates/core/src/workspace_root.rs::find
  - winter-workspace/crates/core/src/index_lookup.rs::answer
  - winter-workspace/crates/core/src/repository_catalog.rs::remove
---

# The layered load of both configuration files

Builds the Load flow's file reading and the Layering of [configuration-model.md](../spec/configuration-model.md), under
`decision:parallel-workspaces/workspace-configuration/layering-policy`, and the Root discovery diagnostic of the same
spec. Delivers the precedence bullets of `product:effective-configuration`, the missing-versus-corrupt bullet of
`product:configuration-state-ownership`, the selection bullet of `product:identify-workspace`, and
`engineering:inert-configuration-read`. The reading of `env_aliases` and `envs_per_workspace` for `ws index` moves onto
the merged document; every other key stays unread, as the spec's Load section allows until a command reads it.

## Change

- `crates/core/src/config.rs` replaces `config_raw.rs`. It reads `.winter/config.toml` and `.winter/config.local.toml`,
  each missing file as an empty document, and ends as `ConfigError::FileRead` with the `Io` or `Toml` cause for a file
  that exists and cannot be read or parsed, naming that file.
- `[agent_model_overrides]` present and not a table, in either file, is `ConfigError::AgentOverridesNotTable` naming the
  file; the variant, and the `ValueType` it carries, are added to `ConfigError` in `termination.rs`.
- One merge function applies the three rules of Layering to the two documents: array append for the two repository
  arrays, a one-level table merge for the seven named tables, and whole replacement for every other key, with each "a
  local value that is not …" case of the rules table. An empty local document returns the shared one unchanged.
- The merge refuses a shared value it cannot read as its rule's kind, as the block under Layering's tables says:
  `ConfigError::ValueType` naming the key and the shared file. The variant is added with `key`, `path`, and `found`, and
  `TypedKey` holding the eight keys the merge can refuse; later slices add the keys and fields they need.
- `env_aliases` and `envs_per_workspace` are read, coerced, and defaulted from the merged document as their rows of Top-
  level keys say, and `EnvsPerWorkspaceTooSmall` is checked there; `ws index` answers exactly as before for every
  configuration it accepted before that none of this epic's refusals touches.
- `ConfigError::RootNotFound` names the start directory and the file discovery looks for, `.winter/config.toml`, so the
  diagnostic says how a workspace is selected. Discovery itself is unchanged.
- `repository_catalog::remove` runs the load once, after root discovery and before any read or edit of the file it
  edits, as check 2 of the order of checks in
  [repository-removal.md](../../../../work/parallel-workspaces/repository-catalog/spec/repository-removal.md) says;
  every refusal a later slice adds to the load then applies to `repo remove` with no further change.
- Existing tests the load contradicts are restructured, not taken as evidence against it: in `tests/repo_remove.rs`,
  `the_lock_is_released_after_a_removal_and_after_each_refusal` (its invalid local file now ends every call as
  `FileRead`) and the `--local` case of `a_file_that_is_not_valid_toml_is_a_file_read_error_and_is_left_byte_equal` (the
  shared file is read first).
- The load runs no external command and writes nothing; `ws index` and `repo remove` each load once per invocation.
- `toml` gains its `preserve_order` feature in `crates/core/Cargo.toml`, so tables keep document order; `Cargo.lock` and
  the core `toml` row of `structure.md` "The Cargo workspace" follow.
- `context/architecture/structure.md`: the `config.rs` row replaces the `config_raw.rs` row.
- The command-file deny-list names the new module as `config::`, not `config`: the scan is a substring match and a bare
  `config` matches "configuration" in `commands/repo/remove.rs`. The entry changes in `COMMAND_FILE_NAMES` in
  `crates/dev/src/architecture.rs`, in its pinned test `the_names_no_command_file_may_hold_are_these_fifteen`, and in
  the "Modules a domain entry stands on" row of `context/architecture/changes.md`.

## Acceptance

- Unit tests of the merge, one per row of the Layering tables: a local repository entry appends after the shared ones; a
  local non-array repository value replaces the shared array; `[git] user = { name = … }` drops the shared `user.email`;
  `[keybindings.bindings]` replaces the shared bindings and keeps `leader` and `timeoutlen`; `[env.feature.vars]`
  replaces every shared feature var and keeps `[env.workspace.vars]`; `[capabilities] service = …` replaces only
  `service`; `[space]` drops the shared `[space.kinds]`; `lint = …` and `git_excludes = …` replace the shared value; a
  local non-table for a merged table leaves the shared table untouched; an empty local document returns the shared
  document unchanged.
- Unit tests of the merge refusal, each `ValueType` naming the key and the shared file: shared `git = "x"` with a local
  `[git]`; shared `project_repository = true` with a local `[[project_repository]]`; shared `tui = 1` with a local
  `tui = "y"`; shared `git = "x"` and `tui = 1` with a local file declaring `tui` before `git` reports `tui`. Shared
  `keybindings = ["ab"]` with a local `[keybindings]` merges as the table `{ a = "b" }` under the local children.
- Unit tests of the file reading: a missing local file and a missing shared file each read as empty; an unreadable file
  is `FileRead` with `Io`, and invalid TOML is `FileRead` with `Toml`, each naming the file; `agent_model_overrides = 1`
  in the local file is `AgentOverridesNotTable` naming the local file.
- Component tests of `ws index` in a `WorkspaceBuilder` workspace: `env_aliases` set only in the local file decides the
  alias index; invalid TOML in the local file exits 1 with a diagnostic naming that file; the tree is unchanged after
  each run.
- Component tests of `repo remove`, each with invalid TOML in the local file: removing a declared shared repository, and
  `repo remove bogus/x`, each exits 1 with the `FileRead` diagnostic naming the local file, not `UnknownKind`, and the
  tree is unchanged.
- A component test from a directory with no workspace above it exits 1 with a diagnostic naming the start directory and
  `.winter/config.toml`.
- `grep -rn config_raw crates context` prints nothing.
- `cargo dev gate full` exits 0, and the recorded `ws index` and `repo remove` scenarios pass unchanged.

## Out of scope

- Refusals of individual keys and tables: `top-level-refusals`, `repository-entries`, `env-bands`, `model-tiers`.
- A command that shows the effective settings: left for later in [epic.md](../epic.md).
