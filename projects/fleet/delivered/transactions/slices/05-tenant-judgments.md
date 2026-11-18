---
name: tenant-judgments
external: github:paul-gross/blizzard#2152
depends-on:
  - store-judgments
---

# Tenant judgments

Hands the tenant-lifecycle rules the store judges back to the domain, on the tenant's ordinary unit, and adds the hub
lock table they lock on. Delivers `engineering:store-refuses-stale` for tenant rename and close, and
`engineering:hub-locks`. Spec [hub-locks.md](../spec/hub-locks.md) and
[store-judgments.md](../spec/store-judgments.md#tenant-rename-and-close).

## Change

- One additive revision creates `hub_locks` and seeds its `IDENTITY` and `TENANT_LIFECYCLE` rows;
  `contracts/store/hub.json` is regenerated.
- The identity lock takes its `hub_locks` row in place of the revision row, still as the identity unit's first
  statement, and the identity unit gains its `hub=` declaration.
- Tenant rename runs in the tenant's ordinary unit, whose opening lock refuses a closing tenant;
  `TenantAdministrationStore.rename` drops its closing-tenant condition.
- Tenant close runs in the tenant's ordinary unit, declares `TENANT_LIFECYCLE`, and refuses `LastTenant` in the domain;
  `TenantTeardownStore.close` stops locking every tenant row and drops its last-tenant refusal.
- Tenant creation opens its identity unit with `TENANT_LIFECYCLE`.
- A change note's `## Upgrade` section tells operators to run `blizzard hub migrate` before starting the new build
  (`bzh:manual-migrations`).

## Acceptance

- Neither rename nor close reads a closing or open-tenant fact to refuse.
- Renaming a closing tenant writes nothing; closing the last open tenant is refused with `LastTenant`; closing a closing
  or deleted tenant answers `UNDERWAY` or `DELETED`; all as today.
- The identity lock is the identity unit's first statement, on its `hub_locks` row.
- A close holds no tenant's row but its own.
- Behavior is unchanged at every route and command, once the store is migrated
  (`engineering:behavior-preserving-landings`).
