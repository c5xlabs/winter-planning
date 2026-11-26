---
name: runner-api-leases
external: github:paul-gross/blizzard#2489
depends-on:
  - runner-api-host
code:
  - blizzard/src/blizzard/runner/api/leases.py::router
  - blizzard/src/blizzard/runner/api/lease_scope.py::authorized_lease
  - blizzard/src/blizzard/runner/api/heartbeat.py::router
  - blizzard/src/blizzard/runner/api/session_end.py::router
  - blizzard/src/blizzard/runner/api/asks.py::router
  - blizzard/src/blizzard/runner/api/attachments.py::router
  - blizzard/src/blizzard/runner/api/artifacts.py::router
  - blizzard/src/blizzard/runner/api/facts.py::router
  - blizzard/src/blizzard/runner/api/chunk_scope.py::resolved_git_commit_scope
  - blizzard/src/blizzard/runner/api/escalations.py::router
  - blizzard/src/blizzard/runner/api/gate_decisions.py::router
  - blizzard/src/blizzard/runner/api/git_commits.py::router
  - blizzard/src/blizzard/runner/api/transcripts.py::router
  - blizzard/src/blizzard/runner/api/transcript_segments.py::router
  - blizzard/src/blizzard/runner/api/analytics.py::router
  - blizzard/src/blizzard/runner/api/chunk_asks.py::router
  - blizzard/src/blizzard/runner/api/chunk_notes.py::router
  - blizzard/src/blizzard/runner/api/finding.py::router
  - blizzard/src/blizzard/runner/api/garden.py::router
  - blizzard/src/blizzard/runner/api/history.py::router
  - blizzard/src/blizzard/runner/api/scope.py::router
---

# Lease and transcript runner routes

Migrates one route family of the runner-API row in spec [Mixed ownership](../spec/migration.md#area-ownership) onto the
resolution mechanism `runner-api-services` built, delivering `engineering:narrow-api-edge`'s route portion under
`decision:adopt-di/declared-cutover` and `decision:adopt-di/adoption-batch-size`.

## Change

- Move these route modules from `RunnerWiring` accessors to declared services: `runner/api/leases.py`,
  `runner/api/lease_scope.py`, `runner/api/heartbeat.py`, `runner/api/session_end.py`, `runner/api/asks.py`,
  `runner/api/attachments.py`, `runner/api/artifacts.py`, `runner/api/facts.py`, `runner/api/chunk_scope.py`,
  `runner/api/escalations.py`, `runner/api/gate_decisions.py`, `runner/api/git_commits.py`, `runner/api/transcripts.py`,
  `runner/api/transcript_segments.py`, and the lease-scoped hub-proxied routes `runner/api/analytics.py`,
  `runner/api/chunk_asks.py`, `runner/api/chunk_notes.py`, `runner/api/finding.py`, `runner/api/garden.py`,
  `runner/api/history.py`, `runner/api/scope.py`, which reach the runner through `authorized_lease` and `HubProxy.of`.
  Each takes its `HubProxy` from the `HubProxies` factory `runner-api-host` registered.
- These are the worker-facing lease routes and the transcript reads; the lease-token checks keep their current order.
- `lease_scope.py` and `chunk_scope.py` are shared route dependencies: this slice narrows them, and the
  `runner-api-control` routes that use `chunk_scope.py` take the narrowed form when they migrate.
- Narrow the shared helpers these modules use to typed arguments; a helper another family still calls keeps its old
  callers.

## Acceptance

- `blizzard:component-test`: each migrated route keeps today's refusals (`503` with its label, `404` unknown or drained
  runner, `503` halted runner) and responses; two-runner tests preserve isolation and shared host data.
- `blizzard:service-test` against the mock hub runs a leased worker session end to end unchanged.
- `blizzard:unit-test`: the every-route walker resolves every migrated route at its actual scope; no migrated module
  reads `RunnerWiring`.
- `blizzard:typecheck` and `blizzard:compat` pass with unchanged wire behavior.
