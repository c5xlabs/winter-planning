---
name: env-teardown
depends-on:
  - registry-writes
  - worktree-git-operations
  - epic:parallel-workspaces/workspace-bootstrap
code:
  - winter-workspace/crates/core/src/files.rs::WriteFiles
  - winter-workspace/crates/core/src/files.rs::OsFiles
---

# Tearing down one environment

Adds `env_destroy::destroy` and `Inventory` of [destroy.md](../spec/destroy.md), Per target, steps 1, 2, 4, and 9 to 13:
the directory and confinement check, the inventory with its ownership test, the dirty refusal, worktree removal through
`git worktree remove` alone, the kept-worktrees stop, the empty-directory removal, the self-exclude strip of
[managed-content.md](../../../../delivered/parallel-workspaces/workspace-bootstrap/spec/managed-content.md), and the
release of `registry-writes`. Delivers `product:destroy-environments` but its preview, the release bullet of
`product:stable-environment-allocation`, `engineering:teardown-evidence-completeness` for the refusal, and
`engineering:worktree-effect-confinement` at removal, under
`decision:parallel-workspaces/environment-lifecycle/teardown-policy` and
`decision:parallel-workspaces/environment-lifecycle/location-confinement`, both registered deviations.

## Change

- `crates/core/src/env_destroy.rs` holds `Options` with `force`, `Inventory`, and `destroy`, which emits `InitEvent`s
  through an `InitSink` and answers `RunEnd`: the target's success, or the ending of a sink that could not write.
- An env directory resolving outside the root fails `UnsafeLocation` with nothing removed; a missing one with a record
  is stripped, released, and reported `env_missing`.
- A declared location must pass `owned` at inventory and again just before its removal; anything else in the env
  directory is a leftover, kept and listed as `env_kept`, whatever `--force` says.
- Without `--force`, any worktree whose `git::clean` is false, a failed probe included, refuses the target before any
  effect.
- A failed removal stops the target with `WorktreesKept`, keeping the directory, the exclude block, and the index.
- `git.rs` gains `worktree_remove`; `WriteFiles` gains `remove_empty_dir`, which `OsFiles` implements with
  `std::fs::remove_dir`; `bootstrap.rs`'s `Action` gains `WorktreeRemoved`, `EnvRemoved`, `EnvMissing`, and `EnvKept`.
- `Options`, `Inventory`, `destroy`, and `worktree_remove` are re-exported by name from `lib.rs` for its tests
  ([destroy.md](../spec/destroy.md), Domain); no command calls `destroy` yet.

## Acceptance

- A component test with real git: a clean environment of two worktrees has both removed by `git worktree remove`, its
  directory removed, its exclude block stripped, and its registry entry released; each project's `<env>` branch stays in
  the source checkout; a sibling environment and `projects/` are unchanged.
- Component tests: a dirty worktree refuses the target with `Dirty` naming it, and nothing is removed; with `--force` it
  is removed; a worktree whose clean probe fails is refused as dirty.
- Component tests: a file, a clone, and a worktree the configuration no longer declares are each kept and named in
  `env_kept`, with and without `--force`; the target succeeds and its index is released.
- A component test: a worktree whose removal fails, as with its `index.lock` held, fails the target with
  `WorktreesKept`, and the directory, the exclude block, and the registry entry stay; a re-run after the lock is gone
  completes the teardown.
- A component test: a declared location replaced by a link after the inventory fails `NotAWorktreeOf` and its link
  target is untouched.
- Component tests: an env directory that is a link out of the root fails `UnsafeLocation` and removes nothing; a
  location whose source checkout is gone is kept as a leftover; a `common_dir` failure on the source checkout fails the
  target with nothing removed; an unlistable env directory fails `InventoryUnreadable`.
- Component tests: a missing env directory with a record releases it and succeeds with `env_missing`
  `"index <i> released"`; without a record it fails `EnvDirectoryMissing`.
- A unit test: `OsFiles::remove_empty_dir` fails on a directory holding an entry and removes nothing.
- `cargo dev gate slice` exits 0.

## Out of scope

- The nested audit, losses, and teardown (steps 3 and 6, and the child entries of step 4) belong to
  `epic:parallel-workspaces/nested-workspaces`; provision teardown (step 7, and `--no-provision-teardown`) to
  `epic:running-environments/resource-provisioning`; the `on_env_destroy` hooks (step 8) to
  `epic:agent-ready-operation/executable-extensions`; `--strict` and `--no-provision-teardown`, with their `Options`
  fields, are registered as [destroy.md](../spec/destroy.md), System, gives.
- The preview of step 5 lands with `teardown-preview`.
