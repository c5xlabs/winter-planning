---
name: runner-settings
depends-on:
  - registry-lifetimes
code:
  - blizzard/src/blizzard/runner/host_config.py::HostConfig.load
  - blizzard/src/blizzard/runner/loop_wiring.py::RunnerStepper._context
  - blizzard/src/blizzard/runner/loop_wiring.py::LoopWiring.of
  - blizzard/src/blizzard/runner/composition.py::_build_runner
  - blizzard/src/blizzard/runner/harness/claude_code/adapter.py::ClaudeCodeAdapter
  - blizzard/src/blizzard/runner/harness/opencode/adapter.py::OpenCodeAdapter
  - blizzard/src/blizzard/runner/harness/harness_telemetry_plan.py::HarnessTelemetryPlan
  - blizzard/src/blizzard/runner/harness/capability_snapshot.py::HarnessVersionCache
  - blizzard/src/blizzard/runner/lifecycle/spawn.py::Spawner
  - blizzard/src/blizzard/wire/lease.py::LeaseView
  - blizzard/src/blizzard/runner/api/wiring.py::bind_runner
  - blizzard/tests/harness/runner-host.ts::reconfigure
---

# A runner's settings, live

Makes every per-runner `pass` and `session` key an entry of the runner's store, read per tick and handed to each
session, with the runner's settings routes and CLI verbs ([runner-store.md](../spec/runner-store.md) §Data, §A write,
§The reader, §Seeding steps 4–5; [runner-api.md](../spec/runner-api.md) §Who may write, §Routes, §Views, §A settings
write, §CLI; [passes.md](../spec/passes.md) §A runner's tick, §The objects settings reach, §Sessions). Delivers
`product:runner-settings-in-store`, `product:runner-settings-api`, `product:runner-file-seeds`,
`product:tick-lived-settings`, `product:session-settings`, `product:runner-settings-all-live`, and the runner half of
`product:settings-in-force`; bound by `engineering:running-session-undisturbed`,
`engineering:unedited-daemon-unchanged`, and `engineering:upgrade-migration`, and shaped by
`decision:config/runtime/runner-scoped-settings` and `decision:config/runtime/pull-per-pass`.

## Change

- `runner_settings`, `runner_setting_entries`, and `runner_setting_changes`, runner-scoped, with their revision; the
  seed of each runner's keys with its `runner_identity` row; `RunnerSettingsReader`.
- `RunnerStepper._context` rebuilt when the revisions move; the worker-scratch sweep run only at start.
- Adapters, the telemetry plan, and health probes take their settings at the call; `HarnessVersionCache` keyed by
  binary; the loop's hub client keyed by its timeout.
- `SessionSettings` handed to each launch; `lease_spawns` gains the two revisions and `transcripts_root`; `LeaseView`
  gains the two revisions.
- `require_superuser`; the runner's settings routes and the `settings show|set|unset|changes` verbs, without `--host`,
  `apply`, or `export`.
- Every per-runner key becomes an entry in this change, so no key is ignored in the file without a way to change it.
- `reconfigure()` writes through the settings API on the running host instead of editing a stopped host's file. Every
  spec that reconfigured a host, `tests/harness/checks/runner/host-capabilities.spec.ts` and
  `tests/service/runner/runner.spec.ts` among them, moves to it; one that only proved the file was re-read at boot is
  deleted; new specs cover what the API promises.
- The operator page on runner configuration gains entries, lifetimes, and the seed; the upgrade note says the file now
  seeds a runner's keys and is not read for them again (`engineering:contracts-follow-code`).
- Context: `bzh:settings-snapshot-per-pass` binding the runner; `bzh:config-read-on-use` binding the runner's entries
  (`engineering:context-rules`).

## Acceptance

- `blizzard:unit-test` over every refusal of [runner-api.md](../spec/runner-api.md) §A settings write, and over the
  seed's three cases, an unset entry surviving a restart against a file stating the key among them.
- `blizzard:component-test` over:
  - a write landing between two ticks taking hold at the second, and a write during a tick not reaching it;
  - a refused write leaving the runner as it was, and two writes racing one revision;
  - lowering `agents` below the running sessions stopping claims and no session;
  - a model alias, autonomy, and worker-environment edit each reaching the next spawn and not a running one;
  - a moved `claude_code.binary` probed at the next refresh;
  - a session's transcript read from the root it launched with after `transcripts_root` moves;
  - a rebuild that does not re-run the scratch sweep.
- `blizzard:service-test` on a stepped host writing `throttle.budget.usd` and stepping the tick.
- `blizzard:harness-check` over the new `reconfigure()`.
- `blizzard:store-contract`, `blizzard:cli-contract`, `web:client-drift`, and `blizzard:compat` pass.
- `blizzard:rehearsal`: an upgraded host seeded from its file runs exactly as before.
- `blizzard:manual-runner`.

## Out of scope

- Host-level keys, `tick_seconds`, and `--host`: [04-host-settings](./04-host-settings.md).
- The local page: [08-local-page](./08-local-page.md).
