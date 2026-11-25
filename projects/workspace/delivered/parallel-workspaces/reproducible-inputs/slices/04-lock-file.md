---
name: lock-file
external: github:paul-gross/winter#217
depends-on:
  - module-scaffold
code:
  - winter-workspace/crates/core/src/workspace_files.rs::CONFIG_FILE
---

# Reading and upserting the lock file

Fills `config_lock.rs` with the reader and upsert of `.winter/config.lock`, as the Domain and Data of
[lock-and-state.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/lock-and-state.md) give them,
and `LockEntry` and `RefKind` of [standalone-pins.md](../spec/standalone-pins.md). Delivers the record half of
`engineering:resolution-write-integrity` through the delivered write integrity, under
`decision:parallel-workspaces/workspace-configuration/write-conflicts`, and the shareable-records bullet of
`product:standalone-reference-policy`: the lock is its own committed file, apart from `state.toml` and the local
configuration. Nothing calls it yet.

## Change

- `workspace_files.rs` gains `LOCK_FILE`, `.winter/config.lock`, beside `CONFIG_FILE`, which reading and upsert use.
- Reading answers an empty lock for a missing file and refuses every other row of the lock's error table as
  `ConfigError::Lock`; a malformed file is never read as absent. `version` is accepted as the integer `1`, the float
  `1.0`, or `true`, as lock-and-state.md gives it. An entry's `name`, `ref`, `kind`, or `commit` that is an integer, a
  float, or a boolean reads as Python's `str()` gives it, as the reference coerces each
  (`modules/workspace/internal/config_lock_repository.py:69-72`); the probe of `update-command` confirms it for an
  integer `commit`.
- Upsert is one `WriteFiles::update` read-modify-write: read, replace the entry by `name`, sort by name, and write the
  managed comment, `version = 1`, and one `[[standalone]]` block per entry with the full commit. A read error inside the
  update closure is carried out through a captured variable and leaves the file unwritten.
- The written bytes are the reference's for the same entries: recorded scenarios compare `.winter/config.lock` byte for
  byte.
- Reading and upsert are re-exported by name from `lib.rs` for their component tests.

## Acceptance

- Unit tests of the serialiser: one entry is written byte for byte as the lock example of lock-and-state.md; two entries
  upserted out of order are written sorted by name; an empty lock writes the comment and `version = 1` alone.
- Unit tests of the version: `1`, `1.0`, and `true` are read; `2`, `"1"`, and an absent `version` are
  `UnsupportedVersion`.
- Unit tests of the coercion: `commit = 123` reads as `"123"`, `ref = 1.5` as `"1.5"`, and `ref = true` as `"True"`.
- Component tests, one per row of the lock's error table: an unreadable file, invalid TOML, `version = 2`, a missing
  `commit`, and `kind = "branchy"`.
- Component tests: an upsert into a malformed lock fails with `ConfigError::Lock` and the file's bytes unchanged; two
  threads upserting different names into one lock leave both entries.
- `cargo dev gate slice` exits 0.
