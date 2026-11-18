# Exclusions in the store

How the two in-process exclusions move into the store, for `store-held-exclusions`. Delivers
`engineering:store-held-exclusions`. Paths are under `blizzard/src/blizzard/hub/`, measured on `master` at `eda353d55`.

## Data

```text
# engineering:store-held-exclusions
chunk dependency changes  -> one keyed lock over all tenant chunks (decision:transactions/cycle-check-lock)
export cursor pass/reset  -> a compare-and-append under the EGRESS_CURSOR hub lock
sweep locks               -> unchanged, in process
```

### The dependency lock

```python
# domain/chunk/ports/dependencies.py                                         decision:transactions/cycle-check-lock
DEPENDENCIES_LOCK = LockKey(LockNamespace.CHUNK_DEPENDENCIES, "all")
```

One `keyed_locks` row per tenant, `(tenant_id, "chunk_dependencies", "all")`, created by the first unit that declares it
(`store/locking.py:17`), so it needs no revision. Every change to chunk dependencies declares it beside its chunk locks;
other chunk writes do not.

| Holder                                                           | Declares today                                   | Declares after                                                 |
| ---------------------------------------------------------------- | ------------------------------------------------ | -------------------------------------------------------------- |
| `DependencyService.declare` (`domain/chunk/dependencies.py:155`) | `cycle_lock`; chunks `[dependent, prerequisite]` | `chunks=[dependent, prerequisite], keys=[DEPENDENCIES_LOCK]`   |
| `GroupService.group` (`domain/operations/queue.py:799`)          | `cycle_lock`; chunks `[survivor, *merged]`       | `chunks=[survivor, *merged], keys=[DEPENDENCIES_LOCK]`         |
| `DeleteService.delete` (`domain/operations/delete.py:116`)       | `cycle_lock`; chunk `[chunk_id]`                 | `chunks=[chunk_id], keys=[DEPENDENCIES_LOCK, *work_item_keys]` |
| `DependencyService.release` (`dependencies.py:179`)              | `cycle_lock` alone, in its own implicit unit     | `begin()`, `keys=[DEPENDENCIES_LOCK]`                          |

The delete's `WORK_ITEM` keys come from a pre-read of its chunk's `hub:` refs, re-checked under the lock
([store-judgments.md](./store-judgments.md#garden-proposals-work-items-and-urgency-moves)).

`release` stops running as its own implicit unit: it takes `transaction: LockedTransaction` and its service opens the
unit. `cycle_lock` leaves `TenantStores` (`composition.py:621,677`) and the three services' constructors, and each
`import threading` it needed goes with its `ast-grep-ignore`.

While the tenant row stays a mutex (`decision:transactions/tenant-row-mutex`), every scoped write in a tenant already
waits on it, the cycle check's read already runs inside the write's unit, and so the keyed lock serializes nothing the
tenant row does not. It is what keeps the cycle check correct once the tenant row is relaxed.

### The export cursor

`egress_cursor` (`store/schema.py:2834`) is hub-wide: no `tenant_id`, an autoincrement id, and the newest row per
dataset is the cursor. `EgressStore` holds unscoped connections (`store/internal/egress_store.py:41`), and its read and
append are separate transactions. The pass lock (`composition.py:1771`) keeps an operator's reset from being overwritten
by the advanced cursor a pass in flight appends: the sweep skips a tick while a reset holds it (`egress/sweep.py:111`),
and the reset blocks on a pass (`egress/reset.py:84`).

A compare-and-set with no lock does not hold on Postgres: `egress_cursor` has no uniqueness a predecessor could key, and
under read committed a pass's conditional insert cannot see a reset's uncommitted row, so both commit and the pass's can
still land newest. The append instead runs under a `hub_locks` row, `HubLock.EGRESS_CURSOR`
([hub-locks.md](./hub-locks.md)), held only for the append, not across the pass:

- A pass reads the newest cursor row's id per dataset when it starts. When it appends, it takes `EGRESS_CURSOR`,
  re-reads the newest id, and appends its advanced cursor only if that id is still the one it started from.
- A reset takes `EGRESS_CURSOR` and appends unconditionally.

```python
# domain/observability/egress/repository.py                                  engineering:store-held-exclusions
class EgressCheckpoint:
    ...
    cursor_id: int | None = None   # the row's id, set on read; EgressCheckpoint carries none today

class IWriteEgressCursor(IReadEgress, Protocol):
    def append_cursor(self, record: EgressCheckpoint, *, after: int | None) -> bool: ...
    # Under EGRESS_CURSOR, inserts when the dataset's newest cursor id is ``after`` (``None``: none yet); False when
    # another append moved it.
    def reset_cursor(self, record: EgressCheckpoint) -> None: ...   # under EGRESS_CURSOR, unconditionally
```

The egress store is hub-wide and has no tenant unit to declare a lock in, so `EgressStore` takes `EGRESS_CURSOR` on the
one short write it opens for the append: the declared exception to `write-entry.md`'s rule that a store never locks a
connection it opened for itself. The sweep no longer skips a tick while a reset runs, and no write is held open across a
pass. A pass that loses has already placed the files and manifest for its range, the same state the crash point between
manifest and cursor leaves today (`egress/sweep.py:63-67`); the next pass places that range again from the reset cursor,
which the delivery semantics already accept.

## Flows

### Two dependency changes that would close a cycle together

1. Each opens `begin()` and declares its chunks with `DEPENDENCIES_LOCK`.
2. The second blocks on the key row until the first commits.
3. The second reads the standing edges, now holding the first's edge, and refuses the cycle.

### A reset during a pass

1. The pass starts and reads the newest cursor id `n`.
2. A reset takes `EGRESS_CURSOR` and appends cursor `n+1`.
3. The pass takes `EGRESS_CURSOR` once the reset commits, re-reads `n+1`, and appends nothing; its next tick starts from
   the reset cursor.
