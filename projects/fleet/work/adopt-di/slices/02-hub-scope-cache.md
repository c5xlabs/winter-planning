---
name: hub-scope-cache
depends-on:
  - registry
code:
  - blizzard/src/blizzard/hub/composition.py::TenantCache
---

# Cached tenant containers

Builds the locked opener, tenant-cache adapter, and lifecycle owners in spec [Scope ownership](../spec/scopes.md),
delivering `engineering:locked-child-container`, `engineering:scope-identity`, and the hub portion of
`engineering:ordered-teardown`. Follows `decision:adopt-di/cached-scopes`; production activation remains with the
request integration.

## Change

- Add the single locked child-opening helper and tenant cache with explicit child ownership.
- Distinguish process-level `TenantBrokers` from each tenant's broker and borrowed from owned resources.
- Add the canonical leaf/bag bridge primitives from spec [Mixed ownership](../spec/migration.md#system).
- Keep the running app on its existing wiring; exercise the new cache with test-owned graph inputs.

## Acceptance

- `blizzard:unit-test` races thirty-two tenant opens/resolutions: one child and one service per tenant; two tenants have
  different stores/brokers; lookup after closure is refused.
- Eviction closes only the selected tenant, failed initialization publishes no entry, repeated close is harmless, and
  borrowed inputs remain open.
- A deterministic close-versus-first-resolution race finalizes any completed resource exactly once and rejects further
  lookup, including through an already-open request child; it does not wait for whole service operations to finish.
- Cross-thread/API-loop/action-loop tests resolve shared services while loop-owned resources construct and close on
  their existing owner; configured clients close before that owner stops and before the shared engine is disposed.
- A mixed leaf/service/bag fixture is acyclic and identity-preserving; a provider depending on its aggregate is
  rejected.
- No production request or background path changes yet.
