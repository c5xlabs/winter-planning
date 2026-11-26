---
name: runner-settings
external: github:paul-gross/blizzard#2129
depends-on:
  - registry-lifetimes
  - settings-record
  - harness-set
code:
  - blizzard/src/blizzard/runner/host_config.py::HostConfig.load
  - blizzard/src/blizzard/runner/loop_wiring.py::RunnerStepper._context
  - blizzard/src/blizzard/runner/loop_wiring.py::LoopWiring.of
  - blizzard/src/blizzard/wire/lease.py::LeaseView
  - blizzard/src/blizzard/runner/api/wiring.py::bind_runner
  - blizzard/tests/harness/runner-host.ts::reconfigure
---

# A runner's settings, live

Makes every per-runner `pass` and `session` key an entry of the runner's store. Entries are read per tick and handed to
each session, and the runner gets its settings routes and CLI verbs ([runner-store.md](../spec/runner-store.md) §Data,
§A write, §The reader, §Seeding step 5; [runner-api.md](../spec/runner-api.md) §Who may write, §Routes, §Views, §A
settings write, §CLI; [passes.md](../spec/passes.md) §A runner's tick, §A rebuild that fails, §Sessions). Delivers:

- `product:runner-settings-in-store`, `product:runner-settings-api`, `product:runner-file-seeds`;
- `product:pass-lifetime`, `product:session-lifetime`, `product:immutable-lifetime`, `product:tick-lived-settings`,
  `product:session-settings`, `product:runner-settings-all-live`;
- the runner half of `product:settings-in-force`, and the runner's side of `product:independent-daemon-settings`.

It is bound by `engineering:running-session-undisturbed`, `engineering:unedited-daemon-unchanged`,
`engineering:upgrade-migration`, and `engineering:off-fleet-wire`, and shaped by
`decision:config/runtime/runner-scoped-settings` and `decision:config/runtime/pull-per-pass`. It serves the runners the
file names, which already have identity rows. The host's own revision and record tables are not built yet, so the reader
treats them as absent ([runner-store.md](../spec/runner-store.md) §The reader).

## Change

- `runner_settings`, `runner_setting_entries`, and `runner_setting_changes`, runner-scoped, with their revision; the
  seed of each file runner's keys through `seed_entry`; `RunnerSettingsReader`.
- The runner's write is the shared `write` with the host file's validation.
- `RunnerStepper._context` rebuilds when the revisions move, and so does the harness set when its keys move. A rebuild
  that fails keeps the last good context and set, shows `in_force.failed`, and is retried each tick. The worker-scratch
  sweep runs only at start.
- `lease_spawns` gains the two revisions (the host's null until 06) and `transcripts_root`, and `LeaseView` gains the
  two revisions.
- `require_superuser`, the runner's settings routes, and the `settings get|set|unset|changes` verbs, without `--host`,
  `apply`, or `export`.
- Every per-runner key becomes an entry in this change, so no key is ignored in the file without a way to change it.
- `reconfigure()` writes a `pass` or `session` key through the settings API on the running host, and keeps editing the
  file and restarting for a `boot` key. Every spec that reconfigures a non-`boot` key moves to the API. A spec asserting
  that a restart takes up an edited file asserts instead that a restart leaves a seeded key as the store holds it.
- The operator page on runner configuration gains entries, lifetimes, and the seed. The upgrade note says the file now
  seeds a runner's keys and is not read for them again (`engineering:contracts-follow-code`).
- Context: `bzh:settings-snapshot-per-pass` binds the runner, and `bzh:config-read-on-use` binds the runner's entries
  (`engineering:context-rules`).

## Acceptance

- `blizzard:unit-test` over every refusal of [runner-api.md](../spec/runner-api.md) §A settings write, `immutable`
  included, and over an unset entry surviving a restart against a file that states the key.
- `blizzard:component-test` over:
  - a write landing between two ticks taking hold at the second, and a write during a tick not reaching it;
  - a refused write leaving the runner as it was, and two writes racing one revision;
  - a write taken while the hub is unreachable, and in force at the first tick that reaches it;
  - lowering `agents` below the running sessions stopping claims and no session;
  - a model alias, autonomy, and worker-environment edit each reaching the next spawn and not a running one;
  - a moved `claude_code.binary` reaching the next spawn through a new harness set while a running session keeps its
    own;
  - a session's transcript read from the root it launched with after `transcripts_root` moves;
  - a deleted prompt file failing the rebuild, the runner claiming under its last context, and the rebuild succeeding
    once the file is back;
  - a rebuild that does not re-run the scratch sweep.
- `blizzard:service-test` on a stepped host against the mock hub:
  - writing `throttle.budget.usd` and stepping the tick;
  - a write to `agents` declared at the next registration;
  - a rename declared at the next registration, with no restart.
- `blizzard:harness-check` over the new `reconfigure()`.
- `blizzard:store-contract`, `blizzard:cli-contract`, `web:client-drift`, and `blizzard:compat` pass.
- `blizzard:rehearsal`: an upgraded host seeded from its file runs exactly as before, and a seeded key edited in the
  file changes nothing after a restart.
- `blizzard:manual-runner`.

## Out of scope

- Host-level keys, `tick_seconds`, and `--host`: [06-host-settings](./06-host-settings.md).
- The local page: [11-local-page](./11-local-page.md).
