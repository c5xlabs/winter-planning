---
name: hosted-runners
depends-on:
  - host-scope-pilot
  - host-settings
  - host-hub-tokens
code:
  - blizzard/src/blizzard/runner/composition.py::HostRunnerReconcile
  - blizzard/src/blizzard/runner/composition.py::build_hosted_runner_service
  - blizzard/src/blizzard/runner/composition.py::build_running_host_runner_service
  - blizzard/src/blizzard/runner/composition.py::_admission_of
  - blizzard/src/blizzard/runner/composition.py::build_runner_admission
  - blizzard/src/blizzard/runner/hosting/service.py::HostedRunnerService
  - blizzard/src/blizzard/runner/cli/runtime.py::StoreHostedRunners
  - blizzard/src/blizzard/runner/cli/runtime.py::_seed_entries
  - blizzard/src/blizzard/runner/cli/runtime.py::_import_token
  - blizzard/src/blizzard/runner/cli/runtime.py::_seed_runner
---

# Hosted runner registrations

Moves the hosted-runners row in spec [Mixed ownership](../spec/migration.md#area-ownership), using the runner flows in
[Scope ownership](../spec/scopes.md#flows). Delivers `engineering:area-wiring` and `engineering:one-instance-per-port`
under `decision:adopt-di/after-runtime-config`.

## Change

- Register `HostRunnerReconcile` and `HostedRunnerService` for the running host at host lifetime, with the running
  host's admission (`build_running_host_runner_service` reaches `build_runner_admission` through `_admission_of`), and
  fill the `app.state.hosted_runners` and `start_failures` slots from them; the host runners routes migrate in
  `runner-api-host`, where they iterate `ServedRunners`.
- Move `StoreHostedRunners` out of the CLI module into the hosting area, reading through the host's engine instead of
  opening one per call.
- The per-runner seed runs on every reconcile, not only at boot: `StoreHostedRunners.built` calls `_seed_entries` for
  each runner it starts. Move `_seed_entries`, `_import_token` and `_seed_runner` into the hosting area as a
  host-lifetime seeder with no `click` dependency. It resolves the registered `HostSettingsReader` and `HubTokens`, and
  the runner settings-store factory, which it takes as a legacy leaf that `runner-stores` adopts.
- Register `RunnerSettingsReaders` at host lifetime
  ([Objects built from settings](../spec/scopes.md#objects-built-from-settings)): one `RunnerSettingsReader` per hosted
  runner, keyed by runner id. The seeder and the host tick read it for any runner, one with no open child included (a
  failed start or a halted runner); `runner-identity-harnesses` resolves the runner child's reader from it. A refusal
  the seed raises as `click.ClickException` today becomes a typed seed refusal that `StoreHostedRunners` turns into the
  same `HaltedRunner` message.
- The seed loop (`seed_host`, `seed_hosted_runners`, `hosted_entries`, `seed_runners`) stays in the runner CLI and calls
  the registered seeder. Its callers migrate with their commands: the stopped-host calls in `tick` (`seed_runners`),
  `add` and `retire` (`seed_hosted_runners`) in `cli-runner-dual-mode`, and the `host` boot's in `cli-runner-host`.
- Keep the stopped-host `HostedRunnerService` that `runner add` and `retire` use, and its admission, on a bounded
  compatibility entrypoint until their commands migrate.

## Acceptance

- `blizzard:component-test`: adding a runner on a running host starts it at the next tick; retiring one drains it while
  its sibling keeps serving; a failed start is retried; a runner the hub retired is drained.
- `blizzard:component-test`: a reconcile that seeds a runner with a bad settings entry or an unopenable token halts it
  with today's message; the seed constructs no `HostSettingsReader` or `HubTokens` of its own.
- `blizzard:unit-test` (a layering guard in `tests/guards`): the hosting area imports no `click` or `runner.cli` module;
  `blizzard:component-test`: the host tick opens no engine of its own.
- `blizzard:component-test`: the tick reads a halted runner's settings through `RunnerSettingsReaders` with no child
  open.
- `blizzard:unit-test` (`tests/unit/runner/cli`): `runner add` and `retire` on a stopped host, and `runner host` boot
  seeding, behave unchanged.
- `blizzard:typecheck` and `blizzard:unit-test` (the host graph/identity guards in `tests/guards`) pass.
