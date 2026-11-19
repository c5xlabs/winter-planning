# Repository catalog engineering requirements

For `epic:parallel-workspaces/repository-catalog`.

## What already holds

- The load validates every repository entry, hand-authored or added, before a catalog command acts:
  [configuration-model.md](../../../delivered/parallel-workspaces/workspace-configuration/spec/configuration-model.md),
  Repository entries, under `product:parallel-workspaces/workspace-configuration/configuration-validation`.
- Every write of a configuration file is one locked, atomic read-modify-write in the file port, so two concurrent edits
  of different repositories both survive: the same leaf, Write integrity, under `engineering:concurrent-state-safety`.
- `repo remove` is built and recorded: `epic:rust-adoption/parity-validation` built it, with its proof, to
  [repository-removal.md](./spec/repository-removal.md) in `crates/core/src/config_edit.rs`,
  `crates/core/src/repository_catalog.rs`, and `crates/core/src/commands/repo/remove.rs` of `winter-workspace`, with the
  scenarios under `scenarios/repo-remove/`. This epic's slices change it only where that leaf now differs from it: an
  array that is not an array of tables is `NotArrayOfTables`, not `NotFound`.

## Brief requirements

- engineering:catalog-effect-separation — The catalog modules start no program; only `ws worktrees --status` starts one,
  git, through the program port. Serves `product:edit-repository-inventory` and `product:inventory-presence`.
- engineering:add-under-lock — `repo add` re-runs its duplicate checks on the files read under the write lock, so of two
  concurrent adds of one source or name exactly one appends. Serves `product:edit-repository-inventory`.
