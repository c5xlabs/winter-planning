---
name: hub-settings
external: github:paul-gross/blizzard#2136
depends-on:
  - registry-lifetimes
  - settings-record
code:
  - blizzard/src/blizzard/hub/config.py::HubConfig.load
  - blizzard/src/blizzard/hub/composition.py::build_services
  - blizzard/src/blizzard/hub/app.py::Sweep
  - blizzard/src/blizzard/hub/app.py::_sweep_interval
  - blizzard/src/blizzard/hub/delivery/action_pass.py::ActionPassDriver
  - blizzard/src/blizzard/hub/domain/observability/forge_status.py::AnnotationReconciler
  - blizzard/src/blizzard/hub/domain/observability/tracing/sweep.py::TraceExportSweep
  - blizzard/src/blizzard/hub/domain/observability/egress/sweep.py::EgressSweep
  - blizzard/src/blizzard/hub/auth/throttle.py::IpThrottle
  - blizzard/src/blizzard/hub/domain/observability/transcripts.py::TranscriptCaps
  - blizzard/src/blizzard/hub/auth/permissions.py::Permission
---

# The hub's settings

Moves the hub's `pass` keys into one hub-wide record, read per pass and per request, with its routes, CLI, apply, and
export ([hub.md](../spec/hub.md) §Domain, §Data, §Interfaces; [settings.md](../spec/settings.md) §Hub). Delivers
`product:hub-settings-record`, `product:hub-pass-settings`, `product:hub-file-seeds`, and
`product:hub-settings-document`, and the hub's side of `product:independent-daemon-settings`. It is bound by
`engineering:one-hub-process`, `engineering:upgrade-migration`, `engineering:unedited-daemon-unchanged`, and
`engineering:off-fleet-wire`, and shaped by `decision:config/runtime/hub-settings-admin-reads`,
`decision:settings-cli-scopes`, and `decision:config/runtime/export-values-only`. Its write and seed are the shared ones
([lifetimes.md](../spec/lifetimes.md) §The shared settings record).

## Change

- `hub_settings`, `hub_setting_entries`, and `hub_setting_changes` in `GLOBAL_TABLES`; `Permission.SETTINGS_ADMIN`; the
  seed in `blizzard hub serve`; `HubSettingsReader`; `HubConfig.load`'s section parsers factored out to validate a
  write.
- Each holder in [hub.md](../spec/hub.md) §Reading per pass takes its snapshot at its pass's start or its request's
  arrival, and an interval write wakes the waiting sweep.
- The `/api/admin/settings` routes, their wire models named `HubSettings*`, and the `blizzard hub admin settings` verbs,
  apply and export among them.
- The operator pages on hub configuration and on config documents; the upgrade note that the file now seeds these keys
  (`engineering:contracts-follow-code`).
- Context: `bzh:configured-record` gains the hub-wide record; `bzh:config-read-on-use` binds the hub's entries and
  narrows its scope sentence on process settings to the `boot` keys; `bzh:settings-snapshot-per-pass` extends to the
  hub; `bzh:config-key-declared` gains the hub's half of its `ConfigView` sentence (`engineering:context-rules`).

## Acceptance

- `blizzard:unit-test`: `SETTINGS_ADMIN` is held by no tenant role.
- `blizzard:component-test` over a sweep's interval write reaching the next pass, a transcript cap write reaching the
  next ingest in every tenant, and the throttle rebuilt.
- `blizzard:service-test` on the controlled clock writing a sweep interval and stepping.
- `blizzard:component-test` over an export leaving out an entry `at_default`, an unset transcript cap among them, and
  that export applied back writing nothing.
- `blizzard:service-test`: a runner of the previous release registers and claims against the upgraded hub, and the fleet
  wire's contract is unchanged.
- `blizzard:store-contract`, `blizzard:cli-contract`, `blizzard:compat`, and `web:client-drift` pass.
- `blizzard:manual-hub`.
