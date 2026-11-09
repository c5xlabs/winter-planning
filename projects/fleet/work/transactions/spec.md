# Transactions spec

## Interfaces

```python
# engineering:one-write-entry
# IHubTransactions.begin opens every multi-repository change. Its single lock declaration names chunks, then keys,
# each sorted; a unit takes no lock any other way.

# The two forms every repository write takes:

def write(..., transaction: Transaction | None = None) -> ...: ...
# joins the caller's unit when given one; opens its own short write when not

def write_under_lock(..., transaction: LockedTransaction) -> ...: ...
# a write correct only under a lock; no default
```

Removed, each folded into the plain method it shadowed or into ordinary repository reads that join the unit:

| Removed                                                                 | Replaced by                                                      |
| ----------------------------------------------------------------------- | ---------------------------------------------------------------- |
| `IChunkExclusiveWrites`                                                 | `IHubTransactions.begin`                                         |
| `ILockedChunkRead`                                                      | Repository reads of a chunk's facts and record, joining the unit |
| `ILockedWorkRefRead`                                                    | Repository reads of work refs, joining the unit                  |
| `IWriteGraphRepository.locked_name`                                     | The plain graph write                                            |
| Every `*_locked` twin                                                   | The plain method, taking `transaction`                           |
| Locked-handle reads of a route, standing edges, a runner's registration | Repository reads joining the unit                                |
| The publisher a scoped write carries                                    | The caller publishing what the unit returns, after commit        |
| The callback a store queues on commit                                   | The caller publishing what the unit returns, after commit        |

## Data

```text
# engineering:store-held-exclusions
dependency-graph cycle check   -> store lock (scope: decision:transactions/cycle-check-lock)
dependency delete and release  -> the same store lock as the cycle check
export cursor pass             -> store lock
sweep locks                    -> unchanged, in process
```

A lock needing a store row no table holds lands as an expansion; a `keyed_locks` row needs no revision.
