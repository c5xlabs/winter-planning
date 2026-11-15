---
name: hub-request-pilot
depends-on:
  - hub-scope-cache
code:
  - blizzard/src/blizzard/hub/api/auth_session.py::resolve_tenant
  - blizzard/src/blizzard/hub/composition.py::build_tenant_stores
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
  - blizzard/src/blizzard/hub/api/findings.py::list_findings
---

# Authenticated request path and finding pilot

Activates the hub graph through spec [Entrypoint resolution](../spec/entrypoints.md) and the representative finding
graph in spec [Mixed ownership](../spec/migration.md#domain). Delivers the hub's `engineering:born-in-container`,
`engineering:every-route-resolves`, `engineering:graph-checked-whole`, and `product:wiring-beside-code` under
`decision:adopt-di/strangler-migration`.

## Change

- Select the tenant with existing auth/admission rules, then attach a thin request child to the cached tenant graph.
- Keep hub-only authentication process-backed; separate admitted identity/scope records from legacy service access.
- Move finding store/read aliases, `FindingExitService`, and `FindingBucketReader` into area wiring; narrow the findings
  API and its required read/label helpers; retain canonical fields for remaining legacy consumers.
- Install route resolution and ownership inventory gates, transitional hub harness rules, and a working wiring/service
  exemplar. New hub services now use area wiring; the root composes areas.

## Acceptance

- Component tests exercise finding reads/writes through real authenticated routes, with unchanged refusal payloads for
  missing identity, mismatched machine/header tenant, missing membership, unknown/closed tenant, and route admission.
- Both synchronous and asynchronous handlers use `inject_sync`, resolve through the admitted request child with a
  synchronous `get()`, and preserve admission-before-resolution.
- Concurrent requests with different principals share only their tenant's services; request completion/cancellation does
  not close the tenant; a background accessor and retained bag resolve the same finding service.
- An exemplar/test-only additional service is registered and consumed through the production request integration by
  editing only its existing area's wiring and consumer, without root/shared-builder changes.
- Whole-graph, every-route, locked-child, alias/transition identity, wiring/import and narrowing guards pass and fail on
  planted violations; unmigrated routes remain supported.
- `blizzard:typecheck`, relevant component tests, and existing finding/auth API tests pass with unchanged wire behavior.
