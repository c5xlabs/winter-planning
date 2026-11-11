---
name: host-settings
depends-on:
  - runner-settings
code:
  - blizzard/src/blizzard/runner/loop/step_passes.py::HostPassStepper
  - blizzard/src/blizzard/runner/loop_wiring.py::PeriodicDriver
  - blizzard/src/blizzard/runner/loop/periodic_pass_driver.py::PeriodicPassDriver
  - blizzard/src/blizzard/runner/tracing/sweep.py::LeaseTraceSweep
  - blizzard/src/blizzard/runner/cli/runtime.py::host_tick_driver
---

# The host's settings, live

Makes the host-level keys entries, evaluated by the host tick and the trace sweep, with `tick_seconds` and
`tracing.sweep_seconds` read live ([runner-store.md](../spec/runner-store.md) §Seeding step 1;
[passes.md](../spec/passes.md) §The loop's interval, §The trace sweep, §The host tick). Delivers
`product:every-pass-evaluates` and `product:tick-interval-setting`; bound by `engineering:unedited-daemon-unchanged` and
`engineering:upgrade-migration`.

## Change

- `host_settings`, `host_setting_entries`, and `host_setting_changes` with `record_kind = "host"`; their seed,
  `tick_seconds` from `BZ_RUNNER_TICK_SECONDS` only when neither file nor store states one.
- The host tick and the trace sweep take their snapshot at their start.
- `tick_seconds` and `tracing.sweep_seconds` read by the drivers' `interval()`. A committed write of either sets every
  driver's own wake signal (`WakeSignals`).
- `lease_spawns.host_settings_revision` written with each spawn.
- The operator page on the runner host's configuration gains the host's entries and their lifetimes. The upgrade note
  moves `BZ_RUNNER_TICK_SECONDS` to `tick_seconds` (`engineering:contracts-follow-code`).
- The host routes of [runner-api.md](../spec/runner-api.md) §Routes and `--host` on the CLI.
- Context: `bzh:config-key-declared` gains the runner's half of the sentence making a `pass` or `session` key read
  through `ConfigView` a finding, now that every runner key of those lifetimes is an entry
  (`engineering:context-rules`).

## Acceptance

- `blizzard:unit-test` over the driver's wake running at once or waiting out the rest, and one driver's wake leaving
  every other driver woken too.
- `blizzard:unit-test` over where `tick_seconds` comes from: the store's entry, else the file's, else
  `BZ_RUNNER_TICK_SECONDS` at the seed, else the default.
- `blizzard:component-test` over a `tick_seconds` write from an hour to a second taking hold without waiting the hour,
  and a tracing write reaching the next sweep.
- `blizzard:rehearsal`: an upgraded host that set `BZ_RUNNER_TICK_SECONDS` ticks at the same interval.
- `blizzard:harness-check`, `blizzard:store-contract`, `blizzard:cli-contract`, `web:client-drift`, and
  `blizzard:compat` pass.
