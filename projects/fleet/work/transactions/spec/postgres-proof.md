# The Postgres proof

What `blizzard:postgres-uow-concurrency` grows to cover, for `postgres-proof`. Delivers
`engineering:postgres-locking-proof`. Paths are under `blizzard/`, measured on `master` at `eda353d55`.

## System

The method is `scripts/check_hub_uow_postgres.py`, run by hand with `BLIZZARD_UOW_POSTGRES_URL` set (blizzard-context
`verification/blizzard/commands/test-tiers/postgres-uow-concurrency.md`). It makes a scratch database
(`tests/rehearsal/stores.py:77`), builds four tables with raw DDL and no foreign keys (`:238`), and runs pairs of
writers in threads, using `pg_blocking_pids` to see one block on the other:

- chunks `a,b` against `b,a`, and keys in opposite orders, each finishing in canonical order;
- the production tenant guard, asserting the second writer blocks at the tenant row (`:180`);
- sixteen named lock-set pairs of production units (`:80`).

No workflow runs it. The upgrade rehearsal (`.github/workflows/rehearsal.yml`, a `[sqlite, postgres]` matrix) runs real
write paths on Postgres with real foreign keys, but no concurrency, tenant close, or drain. None of the four cases below
is covered.

## Flows

Each case runs against the migrated hub schema, not the scratch DDL, so foreign keys are enforced and the production
tenant guard is the one under test.

| Case                                    | Setup                                                                      | Passes when                                                                    | Closes in `gaps.md`                                  |
| --------------------------------------- | -------------------------------------------------------------------------- | ------------------------------------------------------------------------------ | ---------------------------------------------------- |
| The closed-tenant check a write opens with | Writer A opens a scoped write; a close of the same tenant blocks on A's tenant-row lock, or A blocks on the close's | A write that waited on a committed close refuses `TenantClosed` and writes nothing | §Tenancy's migrations and locks under Postgres, first bullet |
| Two closes and the last two open tenants | Two tenants, each closed concurrently; each close takes its tenant's row and then `TENANT_LIFECYCLE` ([hub-locks.md](./hub-locks.md)) | Exactly one close commits; the other refuses `LastTenant` | same section, third bullet |
| The drain under foreign keys            | A tenant seeded with rows in every table of `DRAIN_ORDER` (`src/blizzard/hub/store/internal/tenant_teardown_store.py:28`) | The drain empties every table with no foreign-key failure | same section, second bullet                    |
| Keyed locks of different keys           | Two units in one tenant declaring different keys, with a test guard in place of the tenant-row lock, as `_CompatibleGuard` does | Neither blocks the other; the same key with two writers inserting it fresh still serializes, the loser's insert caught | §Per-key non-serialization of keyed write locks |

The first case is the one the gap ledger expects to fail: a write that blocked on the close re-checks the row once the
close commits, but evaluates its `NOT EXISTS` against the snapshot it took before blocking, so it can still write into
the tenant just closed. If the proof confirms it, the epic fixes it, by re-reading `tenant_deletions` after the lock is
taken or by having the close update the tenant row the guard locks.

The proof stays a focused method, not a Postgres test tier (`gaps.md` §Per-key non-serialization). Each gap it closes
leaves `gaps.md`.

## Open questions

- **Whether the method gains a workflow.** It runs only by hand. A job beside the rehearsal's Postgres service would
  keep it honest, and would need a workflow guard like `tests/guards/test_rehearsal_workflow.py`.
- **What the method checks today.** `gaps.md` says the method checks only pre-existing lock rows, but only two of its
  keys are pre-inserted (`:254`). The sixteen lock-set pairs name keys no row holds yet, so they may already exercise
  the fresh-insert race; this was read from the code, not run.
