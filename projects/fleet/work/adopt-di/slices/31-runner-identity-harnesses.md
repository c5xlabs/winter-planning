---
name: runner-identity-harnesses
depends-on:
  - runner-stores
  - host-settings
  - host-hub-tokens
  - hosted-runners
code:
  - blizzard/src/blizzard/runner/composition.py::_build_runner
  - blizzard/src/blizzard/runner/composition.py::RunnerProcess
  - blizzard/src/blizzard/runner/settings_reader.py::RunnerSettingsReader
  - blizzard/src/blizzard/runner/hub/sealed_tokens.py::RunnerHubTokenSource
  - blizzard/src/blizzard/runner/hosting/retirement.py::HubNoticedRetirement
  - blizzard/src/blizzard/runner/hub/identity_service.py::RetiredRunnerWatch
---

# Runner identity and retained harness lifetimes

Moves the runner-identity/harness row in spec [Mixed ownership](../spec/migration.md#area-ownership), delivering
`engineering:area-wiring` and `engineering:ordered-teardown`; follows `decision:adopt-di/declared-cutover`.

## Change

- Register runner identity/verdict/token/version/flag holders, `RunnerHubTokenSource`, `harness_key`,
  `HubNoticedRetirement`, `RetiredRunnerWatch` (built today per runner over its identity store and identity client),
  clients, executor, retained harness sets and runner tracing at runner lifetime.
- The runner child resolves its `RunnerSettingsReader` from the host's `RunnerSettingsReaders` (`hosted-runners`) by its
  runner id, replacing its daemon construction in `compose_runner` and the standalone tick's in `runner/cli/runtime.py`.
  Its other constructions are the per-runner seed in `_seed_runner`, which `hosted-runners` already moved onto
  `RunnerSettingsReaders`, and the boot preview in the `host` command, which moves in `cli-runner-host`.
- The `ICurrentRunnerIdentity` holder is built eagerly when the runner child opens, so the runner app's middleware
  readers never construct it on the event loop.
- The first harness set and resolved flags are built when the runner child opens, from its `BuiltRunner`, so a bad entry
  still fails the `compose` start step ([Objects built from settings](../spec/scopes.md#objects-built-from-settings)).
- Preserve dynamic harness-set replacement through the existing owner, rather than freezing its current registry in a
  runner-lifetime service; operation snapshots remain operation-local.
- Replace legacy constructors and narrow moved consumers; keep a runtime owner rather than forwarding `RunnerProcess`.

## Acceptance

- `blizzard:component-test` identity/token/harness tests preserve refresh, refused-token, replacement, hub-noticed
  retirement and retained-invocation behavior.
- `blizzard:component-test`: failed runner construction closes acquired clients/executor/sets and reports the same start
  step; normal closure drains work before host disposal.
- `blizzard:component-test`: the runner child's `RunnerSettingsReader` is the instance `RunnerSettingsReaders` holds for
  its runner id; default factories and role keys hold, and no borrowed input is closed twice; `blizzard:typecheck` and
  `blizzard:unit-test` (the runner graph/identity guards in `tests/guards`) pass.
- `blizzard:component-test` runner API tests pass unchanged and unused runner fields/bridges are removed.
