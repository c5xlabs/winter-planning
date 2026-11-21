---
name: host-settings
depends-on:
  - host-infrastructure
  - epic:config/runtime
code:
  - blizzard/src/blizzard/runner/host_settings_reader.py::HostSettingsReader
  - blizzard/src/blizzard/runner/host_settings_reader.py::HostSettingsInForce
  - blizzard/src/blizzard/runner/settings/authoring.py::HostSettingsAuthoring
  - blizzard/src/blizzard/runner/loop/wake.py::WakeSignals
  - blizzard/src/blizzard/runner/loop_wiring.py::host_pass_stepper
  - blizzard/src/blizzard/runner/composition.py::build_host_process
---

# Host settings registrations

Moves the host-settings row in spec [Mixed ownership](../spec/migration.md#area-ownership), using
[Objects built from settings](../spec/scopes.md#objects-built-from-settings). Delivers `engineering:area-wiring` and
`engineering:one-instance-per-port` under `decision:adopt-di/declared-cutover` and
`decision:adopt-di/after-runtime-config`.

## Change

- Register `HostSettingsReader` once at host lifetime with `HostSettingsInForce`, `HostSettingsAuthoring` and
  `WakeSignals`, replacing the daemon's construction in `build_host_process` and the standalone tick's in
  `runner/cli/runtime.py`. The per-runner seed's construction in `_seed_entries`, which runs on every reconcile,
  resolves this reader once `hosted-runners` registers the seeder; `seed_host`'s, which runs before the hosted root
  opens, moves with the `host` command in `cli-runner-host`.
- The standalone graph keeps `HostSettingsReader` present, built from the `HostConfig` each standalone caller supplies
  (`host-scope-pilot`): the CLI `tick` reads the host record and layers it into the runner's settings, as today. Only
  `HostSettingsInForce` and `HostSettingsAuthoring` are registered absent there (`Registry.absent`), since a standalone
  host serves no settings routes and runs no host tick.
- Register `host_pass_stepper` and the timed drivers that read their interval each wait; no provider captures one pass's
  settings into a longer-lived object.
- Fill the `app.state` slots the host settings routes read from the registered services; the routes themselves migrate
  in `runner-api-settings`, as do the settings apply and export, after the record and runner-settings writers they
  drive.
- Remove the `HostProcess` settings fields (`host_settings`, `host_settings_authoring`, `wakes`) and the
  after-construction attachment, narrowing their consumers first: the host serve and tick paths in
  `runner/cli/runtime.py`, `compose_runner` and `_build_runner` in composition, and `_wire_hosted_app`.

## Acceptance

- `blizzard:component-test`: a committed interval write wakes every driver; a host settings write reaches the next host
  tick and no sooner; a failed rebuild keeps the settings in force and reports the failure.
- `blizzard:unit-test` (`tests/unit/runner/cli`): a standalone `runner tick` still layers the host record into the
  runner's settings; resolving `HostSettingsInForce` there raises `ServiceAbsent`.
- `blizzard:component-test`: one `HostSettingsReader` serves the host tick, the routes and the runners of the daemon
  graph; `blizzard:typecheck` and `blizzard:unit-test` (the host graph/identity guards in `tests/guards`) pass.
