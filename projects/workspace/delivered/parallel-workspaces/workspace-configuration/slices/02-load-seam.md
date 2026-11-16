---
name: load-seam
depends-on:
  - config-error-variants
---

# One load of the whole configuration under `ws index`

Builds the modules and the entry of [discovery-and-loading.md](../spec/discovery-and-loading.md) and puts them under
`ws index` in place of the reader of two keys, under
`decision:parallel-workspaces/workspace-configuration/loaded-workspace-value`. Delivers
`engineering:configuration-snapshot` and `engineering:inert-configuration-read` for the keys this change reads.

The reader, the merge, the build, and the types have no caller until `index_lookup` calls `load`, and unused code fails
the gate, so they land in one change with the switch and the removal of `config_raw.rs`. The acceptance checks are
grouped into four stop points, each of which leaves the gate passing.

## Change

- New `crates/core/src/config_file.rs`, `config_merge.rs`, `config_load.rs`, and `config_model.rs` with the signatures
  of the spec's Interfaces and the flow of its Flows. `config_file::read` returns an empty table for a path that is not
  a regular file (`decision:parallel-workspaces/workspace-configuration/non-regular-configuration-path`).
- `config_merge::merge` implements Layering of [configuration-model.md](../spec/configuration-model.md): the three
  rules, the empty local file, and the table-merge rows for a shared value of the wrong type, a shared array of pairs
  among them (`decision:parallel-workspaces/workspace-configuration/element-type-validation`).
- `config_load::load` checks, in each file before the merge, `agent_model_overrides` (`AgentOverridesNotTable`) and then
  the two repository keys (`RepositoriesNotArray`, under
  `decision:parallel-workspaces/workspace-configuration/repository-array-type`), the shared file first, so the merge
  meets no repository value that is not an array.
- `config_model.rs` defines every type of the spec's Domain and `build`, which calls its children in the order of the
  model's Load, steps 1 to 9. It also holds the shared readers of one value the spec's System names, each with a caller
  in this change: the `ConfigValue` of a value, the `ValueType` of a value, the test for an empty form, the reader of a
  string, and the reader of an array of strings. No later change of this epic defines one of them again.
- `config_model/ports.rs` is complete: `base_port`, `ports_per_env`, `env_aliases`, `envs_per_workspace`, and
  `EnvsPerWorkspaceTooSmall`. `config_model/settings.rs` reads three keys completely: `adopt_extensions`, with
  `InvalidChoice`; and, as step 9 of Load, the top-level `git_excludes` and `main_branch`, with `ValueType` as Value
  types states them. For every other key it returns the default. `repositories.rs`, `env_bands.rs`, and `agents.rs` each
  hold the function `build` calls, returning the defaults of the model without reading the table; `repositories.rs` also
  holds the complete prefix collision check.
- The function of `repositories.rs` that `build` calls at step 1 receives, beside the merged table, the workspace
  `main_branch` value, which a repository entry without a `main_branch` of its own takes. `build` reads that value
  before step 1 as the top-level `main_branch` when it is a non-empty string and as `main` otherwise, and still refuses
  a top-level value of another type at step 9, so the order of refusals is as Load states it. The signature is final in
  this change: the change that fills `repositories.rs` edits no line of `config_model.rs`.
- `crates/core/src/index_lookup.rs` calls `config_load::load` and reads the aliases and `envs_per_workspace` from the
  `Workspace`. `crates/core/src/config_raw.rs` is deleted with its tests, whose cases move to `ports.rs`,
  `config_file.rs`, and `config_merge.rs`.
- `crates/core/src/files.rs`: `MemoryFiles` gains an entry that is a regular file whose read fails. Its existing
  `unreadable` entry is not a regular file, so `config_file::read` loads it as absent.
- `crates/core/tests/ws_index.rs`: the test
  `a_local_configuration_file_that_cannot_be_read_is_a_file_read_error_with_the_io_cause` places a directory at the
  local path, which now loads as absent. It is replaced by two tests: a directory at the local path gives the answer of
  the shared file alone; a regular local file without read permission is `FileRead` with the `Io` cause and the path,
  returning early when the user may read any file.
- The name `config_raw` leaves four places, and each gains `config_file`, `config_load`, `config_merge`, and
  `config_model`: the `mod` lines of `crates/core/src/lib.rs`; `COMMAND_FILE_NAMES` of `crates/dev/src/architecture.rs`,
  whose length becomes eighteen, with the test `the_names_no_command_file_may_hold_are_these_fifteen` of the same file
  renamed for the new count and its literal list updated; the row "Modules a domain entry stands on" of
  `context/architecture/changes.md`; and the Modules table of `context/architecture/structure.md`, where the row of
  `config_raw.rs` is replaced by one row for each new module. The facade exports `Workspace`, `WorkspaceConfig`, and the
  types of its fields.
- Files touched: `crates/core/src/config_file.rs`, `config_merge.rs`, `config_load.rs`, `config_model.rs`,
  `config_model/ports.rs`, `config_model/repositories.rs`, `config_model/settings.rs`, `config_model/env_bands.rs`,
  `config_model/agents.rs`, `index_lookup.rs`, `config_raw.rs` (deleted), `files.rs`, `lib.rs`;
  `crates/core/tests/ws_index.rs`; `crates/dev/src/architecture.rs`; `context/architecture/changes.md`,
  `context/architecture/structure.md`.

## Acceptance

The modules and the switch:

- `cargo dev gate slice` passes with `config_raw.rs` deleted, and `cargo scenarios ws-index` and
  `cargo scenarios repo-remove` replay every existing scenario with no change to a recorded file.
- Every existing test passes unchanged, except the one test of `crates/core/tests/ws_index.rs` named above, and the
  tests of `config_raw.rs`, which move.
- A unit test in `index_lookup.rs` runs the lookup over a `ReadFiles` double that counts the reads of each path, and
  asserts that the shared and the local configuration file were each read exactly once.

The rules, as unit tests:

- Over `MemoryFiles`: a directory at the local path loads as if absent; a file that is not UTF-8 and a file that is not
  TOML are each `FileRead` with the `Toml` cause and the path; a regular file whose read fails is `FileRead` with the
  `Io` cause.
- Of `merge`: each row of both Layering tables; a shared `git = "x"` with a local `[git]` is `ValueType` with
  `Table(Git)`; a shared `keybindings = [["leader", "x"]]` with a local `[keybindings]` is `ValueType` with
  `Table(Keybindings)` and the type `Array`; the same shared array with no `keybindings` in the local file merges
  without an error.
- Of `load`: `AgentOverridesNotTable` names the local file when only the local value is not a table;
  `RepositoriesNotArray` carries the key, the file, and the type found for a string in the shared file, for `false` in
  the local file, and names the shared file when both files hold one.
- Of `settings.rs`: `adopt_extensions = "some"` is `InvalidChoice` carrying the value as TOML text; `git_excludes` as a
  string, as `false`, and as an array holding a number is `ValueType` with `GitExcludes`, and an empty array is no
  error; `main_branch = 7` is `ValueType` with `MainBranch`, and `main_branch = 0` and `main_branch = ""` give `main`.

The pinned names:

- A search for `config_raw` over `crates/` and `context/` finds nothing, and `cargo dev gate slice` passes.

The probes:

- A unit test of `config_file` feeds each text of the Data table of the spec to the parser and asserts whether it is
  accepted. The change reports the four results; where one differs from what
  [command-surface.md](../../component-runtime/spec/command-surface.md) says of
  `decision:parallel-workspaces/workspace-configuration/toml-dialect`, the report says so.
- A unit test of `config_model.rs` asserts `ConfigValue.text` for one value of each TOML type.

## Out of scope

- Reading any key the stub functions own.
- Recorded scenarios of the load, which are the change named load-scenarios.
- The singletons, the lock file, and the state file.
