---
name: runner-api-telemetry
external: github:paul-gross/blizzard#2492
depends-on:
  - runner-api-leases
code:
  - blizzard/src/blizzard/runner/api/traces.py::router
  - blizzard/src/blizzard/runner/api/otlp_receiver.py::router
  - blizzard/src/blizzard/runner/api/events.py::router
  - blizzard/src/blizzard/runner/api/usage.py::router
---

# Telemetry and trace runner routes

Migrates one route family of the runner-API row in spec [Mixed ownership](../spec/migration.md#area-ownership) onto the
resolution mechanism `runner-api-services` built, delivering `engineering:narrow-api-edge`'s route portion under
`decision:adopt-di/declared-cutover` and `decision:adopt-di/adoption-batch-size`.

## Change

- Move these route modules from `RunnerWiring` accessors to declared services: `runner/api/traces.py`,
  `runner/api/otlp_receiver.py`, `runner/api/events.py`, `runner/api/usage.py` (`invocation_usage.py` is
  `runner-api-services`' pilot).
- `events.py`'s stream resolves its broker from the runner child before the stream starts.
- The asynchronous handlers (`events_stream` and the OTLP `receive_traces`, `receive_metrics`, `receive_logs`) take
  through `FromDishka` only eagerly built host-lifetime services; their runner-child services come from a synchronous
  dependency ([Entrypoint resolution](../spec/entrypoints.md#interfaces)).
- `otlp_receiver.py` and `usage.py` take `lease_scope.py`'s narrowed form from `runner-api-leases`.
- Narrow the shared helpers these modules use to typed arguments; a helper another family still calls keeps its old
  callers.

## Acceptance

- `blizzard:component-test`: each migrated route keeps today's refusals (`503` with its label, `404` unknown or drained
  runner, `503` halted runner) and responses; two-runner tests preserve isolation and shared host data.
- `blizzard:sse-contract` passes unchanged for the runner event stream.
- `blizzard:unit-test`: the every-route walker resolves every migrated route at its actual scope; no migrated module
  reads `RunnerWiring`.
- `blizzard:typecheck` and `blizzard:compat` pass with unchanged wire behavior.
