---
name: postgres-proof
external: github:paul-gross/blizzard#2164
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
- A nightly workflow runs the proof on a schedule and on demand, never on a pull request or a `master` push, held to
  that by a guard test and following `blizzard:ci-workflows`; the verifiability entry for
  `blizzard:postgres-uow-concurrency` names it.
- `gaps.md` §Per-key non-serialization is corrected: the method already races two fresh inserts of one key.

## Acceptance

- The proof passes on Postgres for each of the four cases.
- The nightly workflow runs the proof, and its guard test fails on a pull-request or push trigger.
