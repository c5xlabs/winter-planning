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

# `repo remove` refuses a repository array that is not an array of tables

Changes the shipped `repo remove`, built by `epic:rust-adoption/parity-validation`, to the outcome table of
[repository-removal.md](../spec/repository-removal.md), Interfaces: an array key holding anything but an array of tables
is `CatalogError::NotArrayOfTables`, where the command now reports `NotFound`. Delivers the `repo remove` half of the
refused-edits bullet of `product:edit-repository-inventory` under
`decision:parallel-workspaces/repository-catalog/refused-edits`, and the removal bullet of the same requirement on a
fixture with checkouts. Lands in one slice under `decision:parallel-workspaces/repository-catalog/direct-landing`.

## Change

- `CatalogError::NotArrayOfTables { kind, path }` is added beside the four existing variants, with the spec's message
  fields.
- The edit closure of `remove_block_with` checks the rows of the outcome table in their order: a file that does not
  exist is `NotFound`; invalid TOML is `ConfigError::FileRead` with `Toml`; an absent array key is `NotFound`; a key
  holding an inline array, empty or not, or any other non-array-of-tables value the load lets through is
  `NotArrayOfTables`, naming the edited file; then the match, `Entangled`, and the write, as now. No refusal writes the
  file.
- Recorded scenarios under `scenarios/repo-remove/`, recorded with `cargo dev record` from the reference tool, the
  executable path given with the task:
  - `inline-array`: `project_repository = [{ url = "…/api.git" }, { url = "…/web.git" }]`,
    `repo remove project/web
    --json`. The reference removes the element; `scenario.toml` gains
    `[deviation] decision =
    "decision:parallel-workspaces/repository-catalog/refused-edits"`, and `expected/` holds
    exit 1, no document, and the tree unchanged.
  - `keeps-checkout`: a remote, its checkout at `projects/api`, and a worktree `alpha/api`; `repo remove project/api`
    changes the configuration file and nothing else in the tree or the git state.
- `crates/core/tests/repo_remove.rs` gains component tests of the new row.

## Acceptance

- Component tests: `project_repository = []`, `= [{ url = "x" }]`, `= "abc"`, `= false`, and `[project_repository]` each
  end as `NotArrayOfTables` naming the file, with the file byte-equal after; with `--local`, the same in the local file
  with the shared file untouched; an absent key is still `NotFound`.
- `cargo scenarios repo-remove` passes every recorded scenario, and `cargo dev gate full` exits 0.
- The `inline-array` record shows the reference's exit 0 and the element removed; replay compares the binary with
  `expected/` and passes.
- The `keeps-checkout` record and replay show `projects/api`, `alpha/api`, their branches, and the worktree list
  unchanged.

## Out of scope

- The same refusal in `repo add`: `add-command`.
