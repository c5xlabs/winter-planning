---
name: runner-settings
depends-on:
  - registry-lifetimes
---

# A runner's settings, live

Makes every per-runner `pass` and `session` key an entry of the runner's store, read per pass and handed to each
session, with the runner's settings routes and CLI verbs ([runner-store.md](../spec/runner-store.md),
[runner-api.md](../spec/runner-api.md), [passes.md](../spec/passes.md) §A runner's tick, §The objects settings reach,
§Sessions). Delivers `product:runner-settings-in-store`, `product:runner-settings-api`, `product:runner-file-seeds`,
`product:tick-lived-settings`, `product:session-settings`, `product:runner-settings-all-live`, and the runner half of
`product:settings-in-force`.

## Change

- The runner-scoped settings tables and their revision, each read through the runner's bound connection; the seed of
  each runner's keys, with its identity row; the reader; the tick's evaluation and the context's rebuild.
- The harness adapters, telemetry plan, and health probes take their settings at the call; the version cache is keyed by
  binary.
- `SessionSettings` is handed to each launch; the spawn records its revisions and `transcripts_root`.
- The superuser check; the runner's settings routes and their `settings` CLI verbs, without `--host`, `apply`, or
  `export`.
- Each per-runner key becomes an entry in this change, so no key is ever ignored in the file without a way to change it.
- The test harness's `reconfigure()` (`tests/harness/runner-host.ts:168`) writes through the settings API on the running
  host instead of editing a stopped host's file. Every spec that reconfigured a host, among them
  `tests/harness/checks/runner/host-capabilities.spec.ts` and `tests/service/runner/runner.spec.ts`, moves to it; any
  that only proved the file was re-read at boot is deleted; new specs cover what the API now promises.
- The operator page on runner configuration gains entries, lifetimes, and the seed; its upgrade note says the file now
  seeds a runner's keys and is not read for them again.
- Context: `bzh:settings-snapshot-per-pass` binding the runner; `bzh:config-read-on-use` binding the runner's entries
  (`engineering:context-rules`).

## Acceptance

- `blizzard:unit-test` over every refusal of a write, and over the seed's three cases, including an unset entry
  surviving a restart against a file stating the key.
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
