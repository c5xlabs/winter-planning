# Hub locks

One table of hub-wide lock rows, for a lock on a concept no tenant owns, for `tenant-judgments`. Delivers
`engineering:hub-locks`. Paths are under `blizzard/src/blizzard/hub/`, measured on `master` at `eda353d55`.

`keyed_locks` cannot hold a hub-wide lock: its rows belong to a tenant (`tenant_id` is in its primary key,
`store/schema.py:1030`). The one hub-wide lock today, the identity lock, is a no-op `UPDATE` of the migration tool's
revision row (`auth/internal/identity_transaction.py:47`), a row that exists for migrations, not for locking.

## Data

```python
# store/schema.py — in GLOBAL_TABLES: "hub_locks",  # locks on concepts above any tenant     engineering:hub-locks
hub_locks = Table(
    "hub_locks",
    metadata,
    Column("name", String, nullable=False),
    PrimaryKeyConstraint("name", name="pk_hub_locks"),
)
```

One additive revision creates the table and seeds one row per hub lock, applied by `blizzard hub migrate`
(`bzh:manual-migrations`, `bzh:store-expand-contract`); `contracts/store/hub.json` is regenerated. The rows are seeded,
never inserted on first use, so taking a hub lock is always one `UPDATE`, and a store that holds no row for a lock has
not been migrated.

## Domain

```python
# domain/kernel/locks.py                                                     engineering:hub-locks
class HubLock(StrEnum):
    IDENTITY = "identity"
    TENANT_LIFECYCLE = "tenant_lifecycle"
    EGRESS_CURSOR = "egress_cursor"              # store-held-exclusions
```

The revision `tenant-judgments` lands seeds `IDENTITY` and `TENANT_LIFECYCLE`; `store-held-exclusions` lands a second
additive revision seeding `EGRESS_CURSOR`.

| Lock               | Taken by                                    | Excludes                                                                                          |
| ------------------ | ------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `IDENTITY`         | Every identity unit, as its first statement | Two identity units deciding a hub-level identity rule at once, such as the last hub administrator |
| `TENANT_LIFECYCLE` | Tenant creation and tenant close            | Two closes each seeing the other's tenant open, and a close missing a tenant being created        |
| `EGRESS_CURSOR`    | The export pass's append and a reset        | A pass's advanced cursor overwriting a reset ([exclusions.md](./exclusions.md))                   |

## Interfaces

```python
# store/locking.py                                                           engineering:hub-locks
def lock_hub(conn: Connection, lock: HubLock) -> None:
    """A no-op UPDATE of the lock's row; raises when no row holds it — the store was never migrated."""

# domain/kernel/transactions.py — a tenant unit may declare a hub lock beside its chunks and keys
class Transaction(Protocol):
    def lock_set(
        self, *, chunks: Sequence[str] = (), keys: Sequence[LockKey] = (), hub: Sequence[HubLock] = ()
    ) -> "LockedTransaction": ...
```

`lock_set` takes chunks, then keys, then hub locks. A tenant unit may declare `TENANT_LIFECYCLE` only: `IDENTITY` is the
identity unit's alone, and `EGRESS_CURSOR` the egress store's.

Tenant creation runs in an identity unit (`TenantAdministrationStore.create`,
`store/internal/tenant_admin_store.py:164`, through `founding`), and an identity unit has no `lock_set`
(`auth/internal/identity_transaction.py:66`). It gains one hub-lock declaration, taken after `IDENTITY` and before any
other statement:

```python
# auth/identity_transaction.py
class IIdentityTransactions(Protocol):
    def begin(self, *, hub: Sequence[HubLock] = ()) -> AbstractContextManager[IdentityWrites]: ...
    # Takes IDENTITY, then each of ``hub`` in order; TENANT_LIFECYCLE is the only one an identity unit may name.
```

Tenant creation opens `begin(hub=[HubLock.TENANT_LIFECYCLE])`.

The identity lock moves onto the table: `_lock_identities` takes `lock_hub(conn, HubLock.IDENTITY)` in place of the
revision-row `UPDATE`, and still as the unit's first statement. Its error for a missing row, and
`tests/component/hub/auth/test_identity_transaction.py::test_the_identity_lock_is_the_units_first_statement`, name the
new row.

## Flows

### Lock order

Every unit that takes more than one of these takes them in one order:

1. `IDENTITY`
2. the rows of tenants that exist: a tenant unit's opening lock
3. `TENANT_LIFECYCLE`

`EGRESS_CURSOR` is taken alone, by a write that takes no other lock.

| Unit             | Takes, in order                                                   |
| ---------------- | ----------------------------------------------------------------- |
| An identity unit | `IDENTITY`, then the rows of the tenants its writes join          |
| Tenant creation  | `IDENTITY`, `TENANT_LIFECYCLE`, then inserts the new tenant's row |
| Tenant close     | the closing tenant's row, `TENANT_LIFECYCLE`                      |

No unit holding a later lock waits on an earlier one, so no two of them wait on each other. Creation holds
`TENANT_LIFECYCLE` while it inserts its new tenant's row, which no other unit can hold, so it waits on no existing
tenant's row and a close holding one cannot deadlock with it.

### Closing a tenant

1. Open the closing tenant's ordinary unit: its opening lock takes the tenant's row, and refuses a tenant already
   closing with `TenantClosed` and one whose row is gone with `UnknownTenant`. On either, the domain reads the tenant's
   deletion: one there answers `UNDERWAY` or `DELETED`, as `close` returns today; none re-raises `UnknownTenant`.
2. Declare `TENANT_LIFECYCLE`.
3. Read the open tenants joining the unit — `tenants` and `tenant_deletions` are global tables — and refuse `LastTenant`
   in the domain when no other is open.
4. Write the tenant's deletion fact.

A close no longer locks every tenant row (`_lock_every_tenant`, `store/internal/tenant_teardown_store.py:45`), so it no
longer holds every tenant's writes while it runs.

### Renaming a tenant

Rename runs in the tenant's ordinary unit. Its opening lock refuses a closing tenant with `TenantClosed`, and the domain
answers that with the `None` rename returns today; the store's rename writes unconditionally. Rename takes no hub lock:
a close racing it waits on the tenant's row.
