---
name: runner-api-host
depends-on:
  - runner-api-services
code:
  - blizzard/src/blizzard/runner/api/host.py::router
  - blizzard/src/blizzard/runner/api/host_records.py::router
  - blizzard/src/blizzard/runner/api/host_runners.py::router
  - blizzard/src/blizzard/runner/api/hub_token.py::router
  - blizzard/src/blizzard/runner/api/federation.py::router
  - blizzard/src/blizzard/runner/api/hub_proxy.py::HubProxy
  - blizzard/src/blizzard/runner/api/health.py::router
  - blizzard/src/blizzard/runner/api/readiness.py::router
  - blizzard/src/blizzard/runner/api/clock.py::host_clock_router
  - blizzard/src/blizzard/runner/api/invariants.py::router
  - blizzard/src/blizzard/runner/api/fleet_summary.py::router
  - blizzard/src/blizzard/runner/api/chunk_detail.py::router
  - blizzard/src/blizzard/runner/api/work_items.py::router
---

# Host and federation runner routes

Migrates one route family of the runner-API row in spec [Mixed ownership](../spec/migration.md#area-ownership) onto the
resolution mechanism `runner-api-services` built, delivering `engineering:narrow-api-edge`'s route portion under
`decision:adopt-di/declared-cutover` and `decision:adopt-di/adoption-batch-size`.

## Change

- Move these route modules from `RunnerWiring` accessors to declared services: `runner/api/host.py`,
  `runner/api/host_records.py` (the host workspace and subscription records), `runner/api/host_runners.py`,
  `runner/api/hub_token.py`, `runner/api/federation.py`, `runner/api/hub_proxy.py`, `runner/api/health.py`,
  `runner/api/readiness.py`, `runner/api/clock.py`, `runner/api/invariants.py`, and the hub-proxied reads
  `runner/api/fleet_summary.py`, `runner/api/chunk_detail.py`, `runner/api/work_items.py`.
- Register a runner-lifetime `HubProxies` factory over the runner's `hub_proxy_client` and config, whose `named(what)`
  returns the `HubProxy` a route names by its module (`what` stays per route, for its failure log), and which keeps
  today's `503` "runner not wired to a hub — start via `blizzard runner host`" when `hub_url` is empty; move this
  family's callers off `HubProxy.of`; `HubProxy.of` keeps its callers in `runner-api-leases` and `runner-api-control`,
  and the last of them, `runner-api-control`, retires it.
- `federation.py`'s routes migrate here; `_gate_web_surface`'s use of `require_human_session` in middleware is
  `runner-api-services`' and resolves only eagerly built host services.
- The host routes that report per runner (environments in `host.py`, the subscription records' spenders in
  `host_records.py`, the runners list in `host_runners.py`, `hub_token.py`) iterate `ServedRunners` in place of
  `RunnerWiring.for_runner`.
- Narrow the shared helpers these modules use to typed arguments; a helper another family still calls keeps its old
  callers.

## Acceptance

- `blizzard:component-test`: each migrated route keeps today's refusals (`503` with its label, `404` unknown or drained
  runner, `503` halted runner) and responses; two-runner tests preserve isolation and shared host data.
- `blizzard:component-test`: a host list route over two runners, one draining, answers with the other.
- `blizzard:unit-test`: the every-route walker resolves every migrated route at its actual scope; no migrated module
  reads `RunnerWiring`.
- `blizzard:typecheck` and `blizzard:compat` pass with unchanged wire behavior.
