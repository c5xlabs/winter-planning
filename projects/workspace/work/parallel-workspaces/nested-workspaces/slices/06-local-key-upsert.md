---
name: local-key-upsert
---

# The local-key upsert writer

Adds the "Upsert local keys" writer of
[configuration-model.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/configuration-model.md),
Writers, which [delegation.md](../spec/delegation.md), The nested step, step 3, applies to a child's
`.winter/config.local.toml`: an edit of the bytes read under the lock that sets each given key and keeps every other
byte. Builds the content-preserving half of `product:nested-local-inheritance` and of
`engineering:delegation-write-integrity`, under
`decision:parallel-workspaces/workspace-configuration/write-byte-preservation`; the TOML is edited as
`engineering:toml-library` gives.

## Change

- `crates/core/src/config_edit.rs` gains the upsert beside the built writers: a same-form value is edited in place,
  another is deleted and re-inserted, a new scalar goes before the first table and a new table at the end; it answers no
  document when the serialised text is unchanged.
- Line endings and a byte order mark are kept as write-byte-preservation gives.
- The upsert is `config_edit::upsert_local_keys` of [delegation.md](../spec/delegation.md), Domain, re-exported by name
  from `lib.rs` for its tests; no command calls it yet.

## Acceptance

- Unit tests: setting `base_port` over an existing integer edits it in place with the comment on its line and every
  other key and comment kept; setting `base_port` where none exists puts it before the first table; setting a `[git]`
  table where none exists puts it at the end; replacing an inline-table value with a standard table re-inserts the key.
- A unit test: setting every key to the value it already holds answers no document.
- Unit tests: a file of CR LF line endings keeps them on every line it held; a byte order mark is kept.
- A unit test: an empty document gains the keys, scalars first.
- `cargo dev gate slice` exits 0.

## Out of scope

- The keys to set, the lock, and the refusals belong to `nested-init-step`.
