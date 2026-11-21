# Repository catalog engineering requirements

For `epic:parallel-workspaces/repository-catalog`.

## What already holds

- The load validates every repository entry, hand-authored or added, before a catalog command acts, and returns them
  typed in the loaded `Workspace`: resolved names, defaulted main branches and standalone paths, `pinned` coerced.
  [configuration-model.md](../../../delivered/parallel-workspaces/workspace-configuration/spec/configuration-model.md),
  Repository entries, and
  [discovery-and-loading.md](../../../delivered/parallel-workspaces/workspace-configuration/spec/discovery-and-loading.md),
  Domain, under `product:parallel-workspaces/workspace-configuration/configuration-validation`. A repository key that is
  not an array is refused there, so without a concurrent writer only an inline array reaches the catalog's
  `NotArrayOfTables`.
- Every write of a configuration file is one locked, atomic read-modify-write in the file port, so two concurrent edits
  of different repositories both survive, and a failed replacement is `ConfigError::FileWrite`:
  [write-path.md](../../../delivered/parallel-workspaces/workspace-configuration/spec/write-path.md), under
  `engineering:concurrent-state-safety`.
- `repo remove` is built and recorded: `epic:rust-adoption/parity-validation` built it, with its proof, to
  [repository-removal.md](./spec/repository-removal.md) in `crates/core/src/config_edit.rs`,
  `crates/core/src/repository_catalog.rs`, and `crates/core/src/commands/repo/remove.rs` of `winter-workspace`, with the
  scenarios under `scenarios/repo-remove/`; `epic:parallel-workspaces/workspace-configuration` put the load before its
  edit and gave a failed write its own kind. This epic's slices change it only where that leaf now differs from it: an
  inline array is `NotArrayOfTables`, not `NotFound`.

## Brief requirements

- engineering:catalog-effect-separation — The catalog modules start no program; only `ws worktrees --status` starts one,
  git, through the program port. Serves `product:edit-repository-inventory` and `product:inventory-presence`.
- engineering:add-under-lock — `repo add` re-runs its duplicate checks on the files read under the write lock, so of two
  concurrent adds of one source or name exactly one appends. Serves `product:edit-repository-inventory`.
