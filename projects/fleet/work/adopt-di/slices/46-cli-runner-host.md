---
name: cli-runner-host
depends-on:
  - cli-runner-dual-mode
code:
  - blizzard/src/blizzard/runner/cli/runtime.py::host
  - blizzard/src/blizzard/runner/cli/runtime.py::seed_host
  - blizzard/src/blizzard/runner/cli/runtime.py::seed_runners
  - blizzard/src/blizzard/runner/cli/runtime.py::init
  - blizzard/src/blizzard/runner/cli/runtime.py::migrate_cmd
---

# Runner host-start command containers

Delivers the runner daemon-start row and the `host`, `init` and `migrate` local verbs in spec
[Entrypoint resolution](../spec/entrypoints.md), serving `engineering:selective-command-imports`,
`engineering:container-only` and `engineering:ordered-teardown`.

## Change

- `runner host` builds the hosted root directly from its command inputs; the root owns its engine and closes it with the
  host container, and the command builds no duplicate process graph.
- Move the boot seed into that start: `seed_host` writes the host record before the hosted root opens, so it runs over a
  command-owned host root that closes first; `seed_hosted_runners`, `hosted_entries` and `seed_runners` then run in the
  hosted root, `seed_runners` calling the seeder `hosted-runners` registered. These are the boot's own seed calls; the
  stopped-host calls in `tick`, `add` and `retire` moved in `cli-runner-dual-mode`. The boot preview's
  `RunnerSettingsReader` comes from `RunnerSettingsReaders`.
- Migrate `init` and `migrate` to command roots over the selected store graph.
- Retire the compatibility entrypoints whose last callers these verbs are: `runner_admission` (`init`'s join),
  `stored_hub_token` (the `host` boot preview and `init`), `build_host_settings_store` (`seed_host` and `_check`), and
  `build_settings_stores` (the `host` command's `_check`, and `hosted_entries` through the boot's `seed_runners`;
  `store_write_guard` and the stopped-host seed calls left it in `cli-runner-dual-mode`).

## Acceptance

- `blizzard:component-test` (`tests/component/runner/cli`): `runner host` boot seeding, store guard refusals,
  tick-seconds warnings and retired-runner warnings are unchanged; `init` and `migrate` keep their outputs and exit
  codes.
- `blizzard:component-test`: a started host builds and disposes one engine; the seed root closes before the hosted root
  opens.
- `blizzard:typecheck`, `blizzard:config-contract` and `blizzard:compat` pass unchanged.
