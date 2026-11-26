---
name: registry-writes
depends-on:
  - allocation-rule
code:
  - winter-workspace/crates/core/src/state_registry.rs::read_in_order
---

# Allocation and release in the state file

Adds `state_registry::allocate` and `state_registry::release` of [allocation.md](../spec/allocation.md), Allocate and
Release: each one `update` of the file port of
[write-path.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/write-path.md) on
`.winter/state.toml`, the read, the decision of `allocation-rule`, and the write all under the one `.winter/` lock.
Delivers `engineering:allocation-integrity`, the concurrent-creation bullet of `product:stable-environment-allocation`,
and the corrupt-state and interruption bullets of `product:environment-state-recovery`; the inline-table write is a
registered deviation under `decision:parallel-workspaces/workspace-configuration/inline-registry-table`, and the TOML is
written as `engineering:toml-library` and `decision:toml-parity` give.

## Change

- `crates/core/src/state_registry.rs` gains `allocate`, `release`, and `AllocateError`; the edit parses the current text
  as the built `read_in_order` does, so a refusal there is `AllocateError::State` and nothing is written.
- `allocate` calls its `moved(from, to)` callback inside the edit, before the edited document is returned, for a `Moved`
  answer alone; `Recorded` returns no document, so the file is not rewritten.
- A new or moved entry, and a release, are edits of the current text as a `toml_edit::DocumentMut`, keeping every other
  key, entry, and comment in its order; an inline or non-table `env_index` comes back as a standard table.
- `allocate` (as `allocate_env`), `release`, and `AllocateError` are re-exported by name from `lib.rs` for its tests
  ([allocation.md](../spec/allocation.md), Domain); no command calls them yet.

## Acceptance

- Component tests over a temporary workspace: `New` adds the name last with the other entries and every other top-level
  key kept; a comment and another top-level key of the old file survive both an `allocate` and a `release`; `Recorded`
  leaves the file's bytes unchanged; an inline `env_index` is written back as a standard table holding its entries and
  the new name; a missing state file is created holding the one entry.
- A component test: for `Moved`, the callback runs once with `(from, to)` while the file still holds `from`, and the
  file holds `to` once `allocate` returns.
- Component tests: a state file that is not UTF-8 or not TOML ends `AllocateError::State` with `FileRead` naming the
  file, and so does an `EnvIndexError` end as `AllocateError::Index`; in each, the file is unchanged.
- A unit test over a failing `WriteFiles`: an `io::Error` of `update` is `AllocateError::State` with `FileWrite`.
- Component tests of `release`: a recorded name answers `Some(index)` and is removed with the others kept; the last
  entry leaves an empty `[env_index]` table; an unrecorded name answers `None` and the file is not rewritten.
- A component test: eight threads allocating eight new names at once over one workspace, each through its own `update`,
  all succeed, the file records all eight, and no two share an index; the next load accepts the file.
- `cargo dev gate slice` exits 0.
