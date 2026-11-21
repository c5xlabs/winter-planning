---
name: hub-scope-cache
depends-on:
  - registry
---

# Cached tenant containers

Builds the locked opener, tenant-cache adapter, and lifecycle owners in spec [Scope ownership](../spec/scopes.md),
delivering `engineering:locked-child-container`, `engineering:scope-identity`, and the hub portion of
`engineering:ordered-teardown`. Follows `decision:adopt-di/cached-scopes`; production activation remains with the
request integration.

## Change

- Add the single locked child-opening helper and tenant cache with explicit child ownership.
- Register process-level `TenantBrokers` and each tenant's production `EventBroker` (`TenantBrokers.for_tenant`, today
  built in `build_services`) as their single owner; hub events reuses both.
- Implement `TenantContainers.open` (cache lookup or open, raising `UnknownTenant`/`TenantClosed`) and
  `TenantContainers.every_open`, and convert a `ScopeClosed` whose owner is a closed tenant into `TenantClosed`.
- Distinguish borrowed from owned resources; the engine, `HubStoreConnections`, platform tracing and `HubSettingsGraph`
  enter as borrowed context values.
- Own the platform-tracing keys: `IPlatformTracer` and the `instrument_client` hook are aliases over the borrowed
  platform-tracing value, registered here once; configured adapters, identity's OAuth client and flow execution's
  `TurnRunner` consume them.
- Add the canonical leaf/bag bridge primitives from spec [Mixed ownership](../spec/migration.md#system).
- Keep the running app on its existing wiring; exercise the new cache with test-owned graph inputs.

## Acceptance

- `blizzard:unit-test` races thirty-two tenant opens/resolutions: one child and one service per tenant; two tenants have
  different stores/brokers; lookup after closure is refused with `TenantClosed`, and `every_open` yields only open
  tenants.
- Eviction closes only the selected tenant, failed initialization publishes no entry, repeated close is harmless, and
  borrowed inputs remain open.
- A deterministic close-versus-first-resolution race finalizes any completed resource exactly once and rejects further
  lookup, including through an already-open request child; it does not wait for whole service operations to finish.
- Cross-thread/API-loop/action-loop tests resolve shared services while loop-owned resources construct and close on
  their existing owner; configured clients close before that owner stops and before the shared engine is disposed.
- A mixed leaf/service/bag fixture is acyclic and identity-preserving; a provider depending on its aggregate is
  rejected.
- No production request or background path changes yet.
