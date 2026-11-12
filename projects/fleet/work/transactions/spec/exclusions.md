# Exclusions in the store

How the two in-process exclusions move into the store, for `store-held-exclusions`. Delivers
`engineering:store-held-exclusions`. Paths are under `blizzard/src/blizzard/hub/`, measured on `master` at `eda353d55`.

## Data

```text
# engineering:store-held-exclusions
chunk dependency changes  -> one keyed lock over all tenant chunks (decision:transactions/cycle-check-lock)
export cursor pass/reset  -> compare-and-set on the newest cursor row; no lock (Proposed)
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

| Holder                                                   | Declares today                                         | Declares after                                      |
| -------------------------------------------------------- | ------------------------------------------------------ | --------------------------------------------------- |
| `DependencyService.declare` (`domain/chunk/dependencies.py:155`) | `cycle_lock`; chunks `[dependent, prerequisite]` | `chunks=[dependent, prerequisite], keys=[DEPENDENCIES_LOCK]` |
| `GroupService.group` (`domain/operations/queue.py:799`)  | `cycle_lock`; chunks `[survivor, *merged]`             | `chunks=[survivor, *merged], keys=[DEPENDENCIES_LOCK]` |
| `DeleteService.delete` (`domain/operations/delete.py:116`) | `cycle_lock`; chunk `[chunk_id]`                     | `chunks=[chunk_id], keys=[DEPENDENCIES_LOCK]`       |
| `DependencyService.release` (`dependencies.py:179`)      | `cycle_lock` alone, in its own implicit unit           | `begin()`, `keys=[DEPENDENCIES_LOCK]`               |

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

**Proposed.** The pass reads the newest cursor row's id per dataset when it starts, and appends its advanced cursor only
if that id is still the newest; a reset appends unconditionally. A pass that loses appends nothing, and its next tick
starts from the reset cursor. This is the compare-and-set form `bzh:write-reestablishes-read` prefers: it takes no lock
and needs no revision.

```python
# domain/observability/egress/repository.py                                  engineering:store-held-exclusions
class EgressCheckpoint:
    ...
    cursor_id: int | None = None   # the row's id, set on read; EgressCheckpoint carries none today

class IWriteEgressCursor(IReadEgress, Protocol):
    def append_cursor(self, record: EgressCheckpoint, *, after: int | None) -> bool: ...
    # Inserts when the dataset's newest cursor id is ``after`` (``None``: none yet); False when another append moved it.
```

A pass places its files before it appends its cursor, so a pass that loses has already placed the files for the range it
covered. The reset that beat it re-exports from an earlier cursor, so the next pass places that range again.

## Flows

### Two dependency changes that would close a cycle together

1. Each opens `begin()` and declares its chunks with `DEPENDENCIES_LOCK`.
2. The second blocks on the key row until the first commits.
3. The second reads the standing edges, now holding the first's edge, and refuses the cycle.

### A reset during a pass

1. The pass starts and reads the newest cursor id `n`.
2. A reset appends cursor `n+1`.
3. The pass's append, conditioned on `after=n`, inserts nothing; the pass ends without moving the cursor.

## Open questions

- **The export cursor's form.** The engineering requirement asks for a store lock; the compare-and-set above meets its
  acceptance with no lock at all. A lock instead takes a `hub_locks` row ([hub-locks.md](./hub-locks.md)) seeded by a
  revision, and holds a write open across a whole pass. Either way, the sweep no longer skips a tick while a reset runs;
  and whether the files a losing pass already placed are harmless beside the re-export is unverified.
