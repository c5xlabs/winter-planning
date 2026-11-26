---
name: cli-runner-dual-mode
external: github:paul-gross/blizzard#2499
depends-on:
  - cli-runner-client
  - runner-api-retirement
code:
  - blizzard/src/blizzard/runner/cli/runtime.py::tick_cmd
  - blizzard/src/blizzard/runner/cli/runners.py::add
  - blizzard/src/blizzard/runner/cli/runners.py::retire
  - blizzard/src/blizzard/runner/cli/token.py::token_set
  - blizzard/src/blizzard/runner/cli/store_guard.py::store_write_guard
  - blizzard/src/blizzard/runner/cli/host_records.py
  - blizzard/src/blizzard/runner/cli/prompt.py::prompt_clear
  - blizzard/src/blizzard/runner/cli/prompt.py::prompt_install
  - blizzard/src/blizzard/runner/cli/runtime.py::seed_host_records
  - blizzard/src/blizzard/runner/cli/runtime.py::read_recorded_host
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
- Migrate the workspace and subscription record verbs (`runner/cli/host_records.py`) and `prompt clear` and `install`
  the same way: through the host's routes while a host answers, and into the stopped host's store from the command root
  while none does. Their shared `_home_record` and `edit_record` read and write through that root in place of
  `read_host_stores` and `build_host_stores`. Split `runner/cli/host_records.py` and `runner/cli/prompt.py`, which
  import `runner.cli.runtime` and `runner.composition` at module level, so the running-host path imports neither.
- Move `seed_host_records` (the record verbs, `add`, and `seed_runners` for `tick` and the `host` boot) and
  `read_recorded_host` (`add`) onto the command root's host stores.
- Move the stopped-host seed calls onto the registered seeder: `tick`'s `seed_runners` and `add`'s and `retire`'s
  `seed_hosted_runners`.
- Migrate `store_write_guard` (`runner/cli/store_guard.py`), which these verbs and the store-write verbs share, off
  `build_runner_store_files(clock).host_roster(...)` onto a `HostRoster` the command root provides; `runners.py`'s and
  `runtime.py`'s other `host_roster` calls move onto it with their commands here and in `cli-runner-host`.
- Retire the compatibility entrypoints only these verbs call: the stopped-host `HostedRunnerService` and its admission,
  and `build_sealing_hub_tokens`.

## Acceptance

- `blizzard:component-test` (`tests/component/runner/cli`) runs each dual-mode verb, the record verbs and `prompt clear`
  and `install` included, against both a running and a stopped host, with unchanged outputs, refusals and exit codes.
- `blizzard:component-test` (`tests/component/runner/cli`): invoker carrier fakes keep precedence; a stopped-host verb
  that fails closes its command root and leaves no open runner child.
- `blizzard:unit-test` fresh-interpreter loaded-module tests (`tests/support/loaded_modules.py`) show the running-host
  path of each verb imports no store or composition module.
- Each graph validates; `blizzard:typecheck`, `blizzard:cli-contract` (the command surface) and `blizzard:compat` pass
  unchanged.
