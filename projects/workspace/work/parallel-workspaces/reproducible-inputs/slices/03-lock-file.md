---
name: lock-file
code:
  - winter-workspace/crates/core/src/termination.rs::ConfigError
---

# Reading and upserting the lock file

Builds `.winter/config.lock` and `LockError` as the Domain and Data of
[lock-and-state.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/lock-and-state.md) give
them, and `LockEntry` and `RefKind` of [standalone-pins.md](../spec/standalone-pins.md). Delivers the record half of
`engineering:resolution-write-integrity` through the delivered write integrity, under
`decision:parallel-workspaces/workspace-configuration/write-conflicts`, and the shareable-records bullet of
`product:standalone-reference-policy`: the lock is its own committed file, apart from `state.toml` and the local
configuration. Nothing calls it yet.

## Change

- `ConfigError` gains `Lock`, carrying `LockError` with its `Load`, `UnsupportedVersion`, and `MalformedEntry` variants.
- Reading answers an empty lock for a missing file and refuses every other row of the lock's error table; a malformed
  file is never read as absent.
- Upsert is one `WriteFiles` read-modify-write: read, replace the entry by `name`, sort by name, and write the managed
  comment, `version = 1`, and one `[[standalone]]` block per entry with the full commit.
- Reading and upsert are re-exported by name from `lib.rs` for their component tests.

## Acceptance

- Unit tests of the serialiser: one entry is written byte for byte as the lock example of lock-and-state.md; two
  entries upserted out of order are written sorted by name; an empty lock writes the comment and `version = 1` alone.
- Component tests, one per row of the lock's error table: an unreadable file, invalid TOML, `version = 2`, a missing
  `commit`, and `kind = "branchy"`.
- A component test: two threads upserting different names into one lock leave both entries.
- `cargo dev gate slice` exits 0.
