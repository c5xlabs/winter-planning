---
name: runner-stores
external: github:paul-gross/blizzard#2484
depends-on:
  - host-infrastructure
  - hosted-runners
code:
  - blizzard/src/blizzard/runner/composition.py::build_stores
  - blizzard/src/blizzard/runner/composition.py::_build_stores
  - blizzard/src/blizzard/runner/composition.py::build_read_stores
  - blizzard/src/blizzard/runner/composition.py::_build_runner
  - blizzard/src/blizzard/runner/cli/runtime.py::read_stores
---

# Runner-bound repository registrations

Moves the runner-stores row in spec [Mixed ownership](../spec/migration.md#area-ownership), delivering
`engineering:area-wiring` and `engineering:one-instance-per-port` under `decision:adopt-di/declared-cutover`.

## Change

- Register bound runner connections and repository/read aliases under each runner child before resolving consumers,
  including the `ExternalUsageStore` `_build_runner` constructs inline for the daemon's runner graph.
- Keep host stores in the host scope; replace old store constructors with canonical resolutions.
- Narrow affected repository consumers and retain only bounded adapter-internal bundles, not a new service locator.
- Adopt the runner settings-store factory the `hosted-runners` seeder takes as a legacy leaf.
- Turn the store factories only runner CLI commands call (`build_settings_stores`, `build_host_settings_store`,
  `build_read_stores`, `build_host_read_stores`, `runner_admission`, `stored_hub_token`, `build_runner_transfer`,
  `build_runner_removal`, `build_runner_store_files`, `build_runner_ingest`, and the inline `HostHubTokenStore` of the
  transfer path) into bounded compatibility entrypoints over the registered providers; the runner command slices
  (`cli-runner-dual-mode`, `cli-runner-host`, `cli-runner-writes`, `cli-runner-local`) retire them, each in the slice
  that migrates its last caller. The running host's admission moves in `hosted-runners`.
- Delete `runner/cli/runtime.py::read_stores`, which nothing calls (`engineering:dead-members-deleted`).

## Acceptance

- `blizzard:store-contract` and `blizzard:component-test` runner store/isolation tests prove no unbound write or
  cross-runner read; two runners share the engine, not stores.
- `blizzard:component-test`: read/write aliases and remaining runtime references share identity; no second owned store
  factory survives in the daemon graph, and each CLI-edge factory resolves the registered provider.
- `blizzard:typecheck`, `blizzard:unit-test` (the runner graph/identity guards in `tests/guards`) and
  `blizzard:component-test` pass; configured runner-id behavior is unchanged.
