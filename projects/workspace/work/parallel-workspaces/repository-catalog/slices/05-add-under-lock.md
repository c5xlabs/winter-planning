---
name: add-under-lock
depends-on:
  - standalone-location
---

# `repo add` checks duplicates again under the write lock

Adds Writing step c of [repository-addition.md](../spec/repository-addition.md): inside the write's lock the load runs
again on the edited file as read and the other file as read then, and checks 5 and 6 run again on that configuration.
Delivers `engineering:add-under-lock` and the concurrent-edit bullet of `product:edit-repository-inventory`, under
`decision:visible-gap-policy` as hardening no caller sees without a concurrent writer.

## Change

- The edit closure `add` hands `WriteFiles::update` loads the configuration from the bytes it is given for the edited
  file and from `ReadFiles::read` for the other file, then runs checks 5 and 6 on it, before step d. A refusal of that
  load ends the command as its `ConfigError`; a duplicate ends it as the variant of check 5 or 6; neither writes the
  file.
- The first load and checks 1 to 6 stay as they are, so an invocation with no concurrent writer ends exactly as before.

## Acceptance

- Unit tests in `repository_catalog.rs` with a `WriteFiles` stand-in whose `update` hands the closure bytes that differ
  from the file the first load read: bytes that already hold the new URL end as `DuplicateUrl` and write nothing; bytes
  with an unrelated new block are appended to, keeping it; bytes that are not valid TOML end as `FileRead` with `Toml`;
  bytes declaring a standalone at the new location end as `DuplicatePath`.
- `cargo scenarios repo-add` passes every scenario unchanged; `cargo dev gate full` exits 0.
