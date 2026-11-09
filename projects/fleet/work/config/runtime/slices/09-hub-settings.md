---
name: hub-settings
depends-on:
  - runner-settings
---

# The hub's settings

Moves the hub's operational settings into one hub-wide record, read per pass and per request, with its routes, CLI,
apply, and export ([hub.md](../spec/hub.md)). Extends the runner's rule to the hub. Delivers
`product:hub-settings-record`, `product:hub-pass-settings`, `product:hub-file-seeds`, and
`product:hub-settings-document`.

## Change

- The hub-wide tables, `SETTINGS_ADMIN`, the seed, and the reader.
- Every holder reads per pass or per request.
- The routes and CLI, apply and export.
- The operator pages on hub configuration and on config documents; the upgrade note that the file now seeds these keys.
- Context: `bzh:configured-record` gains the hub-wide record; `bzh:config-read-on-use` binds the hub's entries and
  narrows its scope sentence on process settings to the `boot` keys; `bzh:settings-snapshot-per-pass` extends its scope
  to the hub; `bzh:config-key-declared` gains the hub's half of its `ConfigView` sentence (`engineering:context-rules`).

## Acceptance

- `blizzard:unit-test`: the permission is held by no tenant role.
- `blizzard:component-test` over a sweep's interval write reaching the next pass, a transcript cap write reaching the
  next ingest in every tenant, and the throttle rebuilt.
- `blizzard:service-test` on the controlled clock writing a sweep interval and stepping.
- `blizzard:store-contract`, `blizzard:cli-contract`, `blizzard:compat`, and `web:client-drift` pass.
- `blizzard:manual-hub`.
