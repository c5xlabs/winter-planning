---
name: runner-api-control
external: github:paul-gross/blizzard#2490
depends-on:
  - runner-api-leases
code:
  - blizzard/src/blizzard/runner/api/control.py::router
  - blizzard/src/blizzard/runner/api/floor.py::host_router
  - blizzard/src/blizzard/runner/api/takeovers.py::router
  - blizzard/src/blizzard/runner/api/requeues.py::router
  - blizzard/src/blizzard/runner/api/selftests.py::router
  - blizzard/src/blizzard/runner/api/ticks.py::router
  - blizzard/src/blizzard/runner/api/flags.py::router
  - blizzard/src/blizzard/runner/api/flag_guard.py::flag_guard
  - blizzard/src/blizzard/runner/api/harness_health.py::router
  - blizzard/src/blizzard/runner/api/dashboard.py::router
  - blizzard/src/blizzard/runner/api/hub_proxy.py::HubProxy.of
---

# Control and status runner routes

Migrates one route family of the runner-API row in spec [Mixed ownership](../spec/migration.md#area-ownership) onto the
resolution mechanism `runner-api-services` built, delivering `engineering:narrow-api-edge`'s route portion under
`decision:adopt-di/declared-cutover` and `decision:adopt-di/adoption-batch-size`.

## Change

- Move these route modules from `RunnerWiring` accessors to declared services: `runner/api/control.py`,
  `runner/api/floor.py`, `runner/api/takeovers.py`, `runner/api/requeues.py`, `runner/api/selftests.py`,
  `runner/api/ticks.py`, `runner/api/flags.py`, `runner/api/flag_guard.py`, `runner/api/harness_health.py`, and
  `runner/api/dashboard.py`, which composes helpers from this family, `runner-api-leases` (asks, escalations, facts) and
  `runner-api-host` (fleet summary, `HubProxies`), all narrowed by then.
- Retire `HubProxy.of`: `dashboard.py` is its last caller.
- `flag_guard.py`'s guard dependency resolves the flag holder it reads from the runner child.
- Narrow the shared helpers these modules use to typed arguments; a helper another family still calls keeps its old
  callers.

## Acceptance

- `blizzard:component-test`: each migrated route keeps today's refusals (`503` with its label, `404` unknown or drained
  runner, `503` halted runner) and responses; two-runner tests preserve isolation and shared host data.
- `blizzard:component-test`: the runner control verbs, which call these routes through `RunnerDaemon`, are unchanged;
  `blizzard:unit-test`: no module calls `HubProxy.of`.
- `blizzard:unit-test`: the every-route walker resolves every migrated route at its actual scope; no migrated module
  reads `RunnerWiring`.
- `blizzard:typecheck` and `blizzard:compat` pass with unchanged wire behavior.
