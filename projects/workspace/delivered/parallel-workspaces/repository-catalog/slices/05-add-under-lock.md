---
name: add-under-lock
external: github:paul-gross/winter#201
depends-on:
  - standalone-location
code:
  - winter-workspace/crates/core/src/config_load.rs::load
---

# `repo add` checks duplicates again under the write lock

Adds Writing step c of [repository-addition.md](../spec/repository-addition.md) and the `config_load::reload` it calls
(the same leaf, Domain): inside the write's lock the load runs again on the edited file as read and the other file as
read then, and checks 5 and 6 run again on that configuration. Delivers `engineering:add-under-lock` and the
concurrent-edit bullet of `product:edit-repository-inventory`, under `decision:visible-gap-policy` as hardening no
caller sees without a concurrent writer.

## Change

- `config_load::reload(files, root, edited, bytes)` runs the checks of `load` in its order from the reads on, with
  `bytes` in place of the read of `edited` and the other configuration file read through `files`; `load` and `reload`
  share every step after the two reads, so the two cannot drift apart. `load` is unchanged for its callers.
- The edit closure `add` hands `WriteFiles::update` calls `reload` with the bytes it is given, then runs checks 5 and 6
  on the `Workspace` it returns, before step d. A refusal of `reload` ends the command as its `ConfigError`; a duplicate
  ends it as the variant of check 5 or 6; neither writes the file.
- The first load and checks 1 to 6 stay as they are, so an invocation with no concurrent writer ends exactly as before.

## Acceptance

- Unit tests of `reload` in `config_load.rs` with `MemoryFiles`: for the bytes the file holds it returns the `Workspace`
  `load` returns; bytes that are not UTF-8 are `FileRead` naming `edited` with `Toml`; a local file the read port gives
  with an invalid key is that key's `ConfigError` while the edited shared bytes are valid; `edited` as the local file
  reads the shared file through `files`.
- Unit tests in `repository_catalog.rs` with a `WriteFiles` stand-in whose `update` hands the closure bytes that differ
  from the file the first load read: bytes that already hold the new URL end as `DuplicateUrl` and write nothing; bytes
  with an unrelated new block are appended to, keeping it; bytes that are not valid TOML end as `FileRead` with `Toml`;
  bytes declaring a standalone at the new location end as `DuplicatePath`; bytes holding `project_repository = "abc"`
  end as `RepositoriesNotArray` and write nothing.
- `cargo scenarios repo-add` passes every scenario unchanged; `cargo dev gate full` exits 0.
