---
name: remove-refusals
depends-on:
  - epic:rust-adoption/parity-validation
code:
  - winter-workspace/crates/core/src/config_edit.rs::remove_repository_block
  - winter-workspace/crates/core/src/config_edit.rs::CatalogError
  - winter-workspace/crates/core/src/repository_catalog.rs::remove_block_with
  - winter-workspace/crates/core/src/repository_catalog.rs::remove
---

# `repo remove` refuses a repository array written inline

Changes the shipped `repo remove`, built by `epic:rust-adoption/parity-validation` and given its load by
`epic:parallel-workspaces/workspace-configuration`, to the outcome table of
[repository-removal.md](../spec/repository-removal.md), Interfaces: an array key holding anything but an array of tables
is `CatalogError::NotArrayOfTables`, where the command now reports `NotFound`. With the load in place, the value that
reaches the row without a concurrent writer is an array written inline; any other value is already
`ConfigError::RepositoriesNotArray` at the load. Delivers the `repo remove` half of the refused-edits bullet of
`product:edit-repository-inventory` under `decision:parallel-workspaces/repository-catalog/refused-edits`, and the
removal bullet of the same requirement on a fixture with checkouts. Lands in one slice under
`decision:parallel-workspaces/repository-catalog/direct-landing`.

## Change

- `CatalogError::NotArrayOfTables { kind, path }` is added beside the four existing variants, with the spec's message
  fields.
- The edit closure of `remove_block_with` checks the rows of the outcome table in their order: a file that does not
  exist is `NotFound`; invalid TOML is `ConfigError::FileRead` with `Toml`; an absent array key is `NotFound`; a key
  holding an inline array, empty or not, or any other value that is no array of tables, is `NotArrayOfTables`, naming
  the edited file; then the match, `Entangled`, and the write, as now. No refusal writes the file. The load before the
  edit, and `FileWrite` for a failed replacement, stay as they are.
- Recorded scenarios under `scenarios/repo-remove/`, recorded with `cargo dev record` from the reference tool, the
  executable path given with the task:
  - `inline-array`: the file holds `project_repository = [{ url = "…/api.git" }, { url = "…/web.git" }]`, and the step
    runs `repo remove project/web --json`. The reference removes the element. The `[deviation]` table of `scenario.toml`
    names `decision:parallel-workspaces/repository-catalog/refused-edits`, and `expected/` holds exit 1, no document,
    and the tree unchanged.
  - `keeps-checkout`: a remote, its checkout at `projects/api`, and a worktree `alpha/api`; `repo remove project/api`
    changes the configuration file and nothing else in the tree or the git state.
- `crates/core/tests/repo_remove.rs` gains component tests of the new row.

## Acceptance

- Component tests: `project_repository = []`, `= [{ url = "x" }]`, and `= [1]` each end as `NotArrayOfTables` naming the
  file, with the file byte-equal after; with `--local`, the same in the local file with the shared file untouched; an
  absent key is still `NotFound`. `= "abc"`, `= false`, and `[project_repository]` end as `RepositoriesNotArray` from
  the load, the file byte-equal.
- Unit tests of the edit closure in `repository_catalog.rs`, over a `WriteFiles` double that hands it bytes other than
  the loaded file's: `project_repository = "abc"` and `[project_repository]` end as `NotArrayOfTables` and write
  nothing.
- `cargo scenarios repo-remove` passes every recorded scenario, and `cargo dev gate full` exits 0.
- The `inline-array` record shows the reference's exit 0 and the element removed; replay compares the binary with
  `expected/` and passes.
- The `keeps-checkout` record and replay show `projects/api`, `alpha/api`, their branches, and the worktree list
  unchanged.

## Out of scope

- The same refusal in `repo add`: `add-command`.
