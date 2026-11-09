---
name: host-settings
depends-on:
  - runner-settings
---

# The host's settings, live

Makes the host-level keys entries, evaluated by the host tick and the trace sweep, with `tick_seconds` and
`tracing.sweep_seconds` read live ([passes.md](../spec/passes.md) §The trace sweep, §The loop's interval). Delivers
`product:every-pass-evaluates` and `product:tick-interval-setting`.

## Change

- The host-level tables and their seed.
- The host tick's and the trace sweep's evaluation at their start.
- `tick_seconds` and `tracing.sweep_seconds` read live, with the wake on write.
- `--host` on the CLI.
- Context: `bzh:config-key-declared` gains the runner's half of the sentence refusing a `pass` or `session` key read
  through `ConfigView`, now that every runner key of those lifetimes is an entry (`engineering:context-rules`).

## Acceptance

- `blizzard:unit-test` over the driver's wake running at once or waiting out the rest.
- `blizzard:component-test` over a `tick_seconds` write from an hour to a second taking hold without waiting the hour,
  and a tracing write reaching the next sweep.
- `blizzard:harness-check`, `blizzard:store-contract`, `blizzard:cli-contract`, `web:client-drift`, and
  `blizzard:compat` pass.
