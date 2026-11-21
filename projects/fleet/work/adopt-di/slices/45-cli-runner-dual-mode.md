---
name: cli-runner-dual-mode
depends-on:
  - cli-runner-client
  - runner-api-retirement
code:
  - blizzard/src/blizzard/runner/cli/runtime.py::tick_cmd
  - blizzard/src/blizzard/runner/cli/runners.py::add
  - blizzard/src/blizzard/runner/cli/runners.py::retire
  - blizzard/src/blizzard/runner/cli/token.py::token_set
  - blizzard/src/blizzard/runner/cli/store_guard.py::store_write_guard
---

# Runner dual-mode command containers

Delivers the runner dual-mode boundary in spec [Entrypoint resolution](../spec/entrypoints.md), and the local command
root in [Scope ownership](../spec/scopes.md#interfaces), serving `engineering:selective-command-imports`,
`engineering:container-per-process` and `engineering:ordered-teardown`.

## Change

- Add the local command root the runner command slices share: a local verb builds the host's `PROCESS`/`RUNNER`
  translation as a root the command owns, opens the runners it selects through `HostRunnerSet`, and closes the root on
  exit, including exceptions. That is the command's one container: its command-local services register at the host
  translation's `PROCESS` lifetime ([Interfaces](../spec/scopes.md#interfaces)).
- Migrate `tick`, `add`, `retire` and `token set`: the client path while a host answers, and the selected store graph
  from that command root under `store_write_guard` while none does. The standalone `tick` uses the host root
  `host-scope-pilot` gave it.
- Move the stopped-host seed calls onto the registered seeder: `tick`'s `seed_runners` and `add`'s and `retire`'s
  `seed_hosted_runners`.
- Migrate `store_write_guard` (`runner/cli/store_guard.py`), which these verbs and the store-write verbs share, off
  `build_settings_stores` onto the command root's settings stores.
- Retire the compatibility entrypoints only these verbs call: the stopped-host `HostedRunnerService` and its admission,
  and `build_sealing_hub_tokens`.

## Acceptance

- `blizzard:component-test` (`tests/component/runner/cli`) runs each dual-mode verb against both a running and a stopped
  host, with unchanged outputs, refusals and exit codes.
- `blizzard:component-test` (`tests/component/runner/cli`): invoker carrier fakes keep precedence; a stopped-host verb
  that fails closes its command root and leaves no open runner child.
- `blizzard:unit-test` fresh-interpreter loaded-module tests (`tests/support/loaded_modules.py`) show the running-host
  path of each verb imports no store or composition module.
- Each graph validates; `blizzard:typecheck`, `blizzard:cli-contract` (the command surface) and `blizzard:compat` pass
  unchanged.
