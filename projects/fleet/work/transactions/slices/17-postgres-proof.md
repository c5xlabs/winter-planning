---
name: postgres-proof
depends-on:
  - tenant-judgments
---

# Postgres proof

Grows `blizzard:postgres-uow-concurrency` over what SQLite cannot show. Delivers `engineering:postgres-locking-proof`.
Spec [postgres-proof.md](../spec/postgres-proof.md).

## Change

- The proof covers the closed-tenant check a scoped write opens with, the tenant lifecycle lock that keeps two closes
  from closing the last two open tenants, the drain's delete order under enforced foreign keys, and keyed locks of
  different keys not excluding each other.
- Each gap the proof closes is removed from `gaps.md`; a mechanism found broken is fixed here.
- The proof stays a focused method, not a Postgres test tier.

## Acceptance

- The proof passes on Postgres for each of the four cases.
