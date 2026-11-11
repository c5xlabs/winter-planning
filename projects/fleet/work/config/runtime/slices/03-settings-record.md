---
name: settings-record
code:
  - blizzard/src/blizzard/hub/config.py::HubConfig.to_toml
---

# The shared settings record

Adds `foundation/settings_record/`, the write, seed rule, and change row both daemons' settings go through, and moves
the TOML emitter beside it ([lifetimes.md](../spec/lifetimes.md) §The shared settings record, §Seed). Pre-work for
`product:runner-settings-in-store` and `product:hub-settings-record`, and the `validate` step
`engineering:paired-settings-checked`'s checks go into. It changes no operator surface.

## Change

- `SettingsRecord`, `write`, and `seed_entry` in `foundation/settings_record/`, with no table of their own and nothing
  calling them yet.
- `HubConfig.to_toml`'s emitter moved to `foundation/toml_emit.py`. The hub's export is unchanged.

## Acceptance

- `blizzard:unit-test` over `write` with an in-memory record: a stale or missing `If-Match` refused, a refused
  validation committing nothing, a write that moves nothing bumping no revision and appending no change, and an unset
  written `at_default`.
- `blizzard:unit-test` over `seed_entry`'s three cases: an entry present (an `at_default` one included) left alone, a
  key the file does not state left unwritten, a stated key written with door and actor `seed`.
- `blizzard:unit-test` and `blizzard:component-test` pass over the hub's export, the tests changing only their imports.
- `blizzard:structural-gate` passes for the new packages' layering.
