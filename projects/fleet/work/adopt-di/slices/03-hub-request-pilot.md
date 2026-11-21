---
name: hub-request-pilot
depends-on:
  - hub-scope-cache
code:
  - blizzard/src/blizzard/hub/api/auth_session.py::resolve_tenant
  - blizzard/src/blizzard/hub/composition.py::build_tenant_stores
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
  - blizzard/src/blizzard/hub/api/findings.py::list_findings
  - blizzard/src/blizzard/hub/api/auth_session.py::refuse_closed_tenant
  - blizzard/src/blizzard/hub/app.py::create_app
  - blizzard/src/blizzard/hub/app.py::create_app_for_export
  - blizzard/src/blizzard/hub/api/hub_level.py::hub_services
---

# Authenticated request path and finding pilot

Activates the hub graph through spec [Entrypoint resolution](../spec/entrypoints.md) and the representative finding
graph in spec [Mixed ownership](../spec/migration.md#domain). Delivers `engineering:container-per-process` for the hub
and the hub's `engineering:born-in-container`, `engineering:every-route-resolves`, `engineering:graph-checked-whole`,
`engineering:wiring-modules-apart`, `engineering:explicit-factory-for-defaults`, and `product:wiring-beside-code` under
`decision:adopt-di/strangler-migration` and `decision:adopt-di/closed-tenant-request`.

## Change

- Install the tenant-route sequence in spec [Entrypoint resolution](../spec/entrypoints.md#interfaces):
  `process_request` attaches the borrowed process resolver; the synchronous `resolve_tenant` selects the tenant with
  existing auth/admission rules, resolving `OpenTenants` (a legacy leaf here) from it, opens the tenant through
  `TenantContainers.open` and maps `UnknownTenant`/`TenantClosed` to today's `404`; the synchronous `tenant_request`
  then attaches a thin request child to the cached tenant graph; `tenant_bound` stays after it, and `_route_admission`
  still finds `TenantBoundAdmitting` among the route's dependencies.
- An asynchronous handler takes through `FromDishka` only process services the root builds eagerly at startup; a lazily
  built or tenant-scoped service it takes from a synchronous dependency, so first construction and any wait on a child
  container's lock run in the threadpool, never on the event loop.
- A resolution refused because its tenant closed raises `TenantClosed`, answered by the existing `refuse_closed_tenant`
  handler.
- Keep hub-only authentication process-backed through `process_request`; separate admitted identity/scope records from
  legacy service access.
- Register the process clock (`resolve_clock`'s choice, with `SystemMonotonicClock`) the pilot's `FindingExitService`
  reads; the `ControlledClock` control routers stay with hub lifecycle.
- Give the store-free app (`create_app` without services, `create_app_for_export`) `Registry.absent` registrations with
  the `hub_store_unavailable` label, so its whole graph validates; install the hub app-level `ServiceAbsent` handler,
  which answers `hub_store_unavailable_problem`'s `503`, so its routes keep their refusal while taking plain
  `FromDishka[T]`.
- Move finding store/read aliases, `FindingExitService`, and `FindingBucketReader` into area wiring; narrow the findings
  API and its required read/label helpers (`QueryProject`, `PathProject`, `ProjectLabels.read`); retain canonical fields
  for remaining legacy consumers. `IReadProjectRepository`, which those helpers read, enters as a legacy leaf provider
  that projects adopts.
- A port consumed before its owning slice adopts it enters as a legacy leaf provider under that port's key, filled from
  the existing construction; its owner later replaces the leaf under the same key. Every later hub slice follows this.
- While the request bridge is live, tenant deletion evicts and closes the tenant's container entry together with the
  legacy `TenantCache` entry.
- Retarget the hand-wiring gates to the container: `_COMPOSITION_ROOT_FILES`, `_COMPOSITION_ROOTS` and the
  constructed-once check in `tests/guards/test_layering.py`, `_CLOCK_COMPOSITION_ROOTS` in
  `tests/guards/test_clock_injection.py`, `test_hub_store_wiring_goes_through_composition`, and
  `tests/component/hub/test_core_sharing.py`.
- Install route resolution and ownership inventory gates, transitional hub harness rules, and a working wiring/service
  exemplar. The `bzh:domain-core` rule gains the `engineering:dishka-at-edge` boundary (no dishka import below wiring)
  in blizzard-context with this change. New hub services now use area wiring; the root composes areas.

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
- Deleting a tenant leaves no tenant child, closes its configured clients exactly once, and refuses a later request.
- A request whose tenant is deleted between admission and a lazy resolution answers `refuse_closed_tenant`'s `404`, or
  `401` to a runner, never a `500`.
- The store-free app builds with whole-graph validation, and its store-backed routes answer `hub_store_unavailable`
  `503` with the payload the refusal inventory guards pin.
- Tenant admission's cache miss, and first construction for an asynchronous handler's service, run in the threadpool,
  never on the event loop; a failed tenant open answers `404` to every caller, a runner included.
- Log context and tenant span binding by `tenant_bound`, and `TenantBoundAdmitting` admission, are unchanged.
- The transitional `bzh:` rules land in blizzard-context with this change and pass `blizzard-context:reference-lint` and
  `blizzard-context:registry-drift`; the wiring/service exemplar passes `blizzard:typecheck`, `blizzard:unit-test` and
  `blizzard:component-test`.
- `blizzard:typecheck`, `blizzard:component-test`, and existing finding/auth API tests pass with unchanged wire
  behavior.
