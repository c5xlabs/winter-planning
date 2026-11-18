# The one write entry

The target form every hub write takes, how the legacy seam folds into it, and the transitional form that lets each area
converge on its own. Delivers `engineering:one-write-entry`; [areas.md](./areas.md) assigns each piece to its slice.
Paths are under `blizzard/src/blizzard/hub/`, measured on `master` at `eda353d55`.

## Interfaces

### Opening a write

```python
# domain/kernel/transactions.py — keys typed as LockKey (§Lock keys)       engineering:one-write-entry
class IHubTransactions(Protocol):
    def begin(self) -> AbstractContextManager[Transaction]: ...

class Transaction(Protocol):
    # One declaration per unit, before any statement: chunks sorted, then keys sorted by (namespace, key).
    def lock_set(
        self, *, chunks: Sequence[str] = (), keys: Sequence[LockKey] = (), hub: Sequence[HubLock] = ()
    ) -> LockedTransaction: ...
    # hub: a lock on a concept above any tenant (hub-locks.md); a tenant unit may declare TENANT_LIFECYCLE only.
```

A decision that must win exactly once opens `begin()`, declares every chunk and key it reads to decide in one
`lock_set`, and hands the `LockedTransaction` to every participant. No unit takes a lock any other way: a store method
never calls `lock_keys` or `lock_chunk_row` on a connection it opened for itself, or mid-unit on one it was handed. Two
exceptions are declared: the identity unit's hub locks ([hub-locks.md](./hub-locks.md)), and the egress store's
`EGRESS_CURSOR`, which has no tenant unit to declare it in ([exclusions.md](./exclusions.md)).

### How a repository write says whether it needs a lock

A write keeps its own name; its `transaction` parameter says whether it needs a lock.

```python
def record_pause(chunk_id, *, paused, by, at, transaction: Transaction | None = None) -> int: ...
# Correct alone. Called without a unit, it runs in its own implicit unit: one scoped write that takes the tenant row,
# commits when the method returns, and rolls back when it raises (store/transactions.py connection_in).

def record_stop(chunk_id, *, by, at, transaction: LockedTransaction) -> LeverWrite: ...
# Correct only under a lock: required, no default. Its body resolves the connection through
# connection_for(..., chunk=chunk_id), which refuses a unit that did not lock that chunk.
```

No method name carries the lock: no `*_locked`, no `_under_lock`. The signature tells the caller a lock is needed; the
body's `connection_for(..., chunk=)` or `(key=)` checks that the unit locked the right chunk or key.

A repository read a locked decision depends on takes `transaction: Transaction | None = None` and joins the unit when
given one, so the decision reads what it writes under.

### Lock keys

Every key lock names its namespace from one enum, so a namespace cannot be misspelled, two cannot collide unseen, and
every one in use is listed in one place.

```python
# domain/kernel/locks.py                                                     engineering:one-write-entry
class LockNamespace(StrEnum):
    CADENCE_ERRAND = "cadence_errand"
    CHUNK_DEPENDENCIES = "chunk_dependencies"      # new: the dependency lock, decision:transactions/cycle-check-lock
    FINDING = "finding"                            # new: garden-and-work-item-judgments
    FLOW = "flow"
    FORGE = "forge"                                # new: config-judgments
    GARDEN_PROPOSAL = "garden_proposal"            # new: garden-and-work-item-judgments
    GRAPH_NAME = "graph_name"
    PROJECT_SOURCE_LINKS = "project-source-links"
    QUEUE_RANKING = "queue_ranking"
    SECRET = "secret"                              # new: config-judgments
    TENANT_SETTINGS = "tenant_settings"
    TRANSCRIPT_CHUNK = "transcript_chunk"
    TRANSCRIPT_LANE = "transcript_lane"
    URGENCY = "urgency"
    URGENCY_ORDER = "urgency_order"
    WORK_ITEM = "work_item"                        # new: garden-and-work-item-judgments
    WORK_REF = "work_ref"

@dataclass(frozen=True, order=True)
class LockKey:
    namespace: LockNamespace
    key: str

# domain/kernel/transactions.py
class Transaction(Protocol):
    def lock_set(
        self, *, chunks: Sequence[str] = (), keys: Sequence[LockKey] = (), hub: Sequence[HubLock] = ()
    ) -> "LockedTransaction": ...
    def lock_keys(self, namespace: LockNamespace, keys: Sequence[str]) -> "LockedTransaction": ...
    def lock_chunks(self, chunk_ids: Sequence[str]) -> "LockedTransaction": ...   # stays: lock_set(chunks=...)

# store/transactions.py
def connection_for(store, transaction, *, chunk: str | None = None, key: LockKey | None = None) -> Connection: ...
```

The existing values are the strings in use today, so the `keyed_locks` rows already written keep serving and nothing
migrates; the six marked new have no rows yet, and their first unit creates each. `LockKey` orders by namespace, then
key, which is the order `lock_set` takes keys in today. The per-namespace constants (`URGENCY_LOCK_NAMESPACE`,
`ERRAND_LOCK_NAMESPACE`, `TRANSCRIPT_*_LOCK_NAMESPACE`, `TENANT_SETTINGS_LOCK_NAMESPACE`,
`URGENCY_ORDER_LOCK_NAMESPACE`) give way to the enum, and every builder returns a `LockKey`: `urgency_lock`,
`urgency_order_lock`, and `tenant_settings_lock` (`domain/urgency/ports.py:31-39`), `errand_lock`
(`domain/cadence/ports.py:22`), `mint_lock_keys` (`domain/cadence/mint.py:112`), `transcript_lane_lock` and
`transcript_chunk_lock` (`domain/observability/transcripts.py:56,61`), the urgency default set's `lock_keys`
(`domain/urgency/services.py:469`), `READY_RANKING_LOCK` and `BACKLOG_RANKING_LOCK`, and config apply's key set.
`scripts/check_hub_uow_postgres.py` consumes these as tuples (`:57-95`) and converts in the same change; it runs only by
hand and nightly, so the change runs it before landing.

Every key a unit declares is built in the domain, beside the port that needs it. Four builders are store-side only today
and move:

| Namespace              | Today                                                             | Builder moves to                  |
| ---------------------- | ----------------------------------------------------------------- | --------------------------------- |
| `WORK_REF`             | `_POINTER_LOCK_NAMESPACE` and `pointer_lock_key`, store-side      | `domain/chunk/ports/work_refs.py` |
| `PROJECT_SOURCE_LINKS` | `PROJECT_SOURCE_LINKS_LOCK_NAMESPACE`, `project_link_store.py:69` | `domain/projects/links.py`        |
| `GRAPH_NAME`           | `_GRAPH_NAME_LOCK_NAMESPACE`, `graph_store.py:264`                | `domain/graph/model.py`           |
| `FLOW`                 | `_FLOW_LOCK_NAMESPACE`, `flow_store.py:69`                        | `domain/delivery/ports/flows.py`  |

The enum, `LockKey`, and the retyped `lock_set` land in one change that converts every existing namespace and builder,
in `config-judgments`, the first slice to declare a new key. The four store-side builders then move with the slice that
declares them: `FLOW` in `flow-judgments`, `WORK_REF` and `PROJECT_SOURCE_LINKS` in `converge-record-and-work-refs`,
`GRAPH_NAME` in `converge-graph`.

### Removed

Each removal folds into the plain method it shadowed, or into an ordinary repository read joining the unit.

| Removed                                                                                                                                                                                                                                                                                                 | Replaced by                                                                     |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| `IChunkExclusiveWrites.locked`                                                                                                                                                                                                                                                                          | `IHubTransactions.begin` + `lock_set(chunks=, keys=)` — what it already runs    |
| `IChunkExclusiveWrites.locked_work_refs`                                                                                                                                                                                                                                                                | `begin` + `lock_set(keys=)` over the domain-built `work_ref` and link keys      |
| `IWriteGraphRepository.locked_name`, `ILockedGraphNameRead`                                                                                                                                                                                                                                             | `begin` + `lock_set(keys=[graph_name_lock(name)])`                              |
| `ILockedChunkRead`, `ILockedWorkRefRead`                                                                                                                                                                                                                                                                | The reads below, joining the unit                                               |
| `ChunkExclusiveWrites`, `LockedChunkTransaction`, `LockedWorkRefTransaction`, `LockedGraphName`                                                                                                                                                                                                         | Nothing                                                                         |
| `ChunkHandle`, `WorkRefHandle`, `chunk_handle_connection`, `work_ref_handle_connection`                                                                                                                                                                                                                 | `connection_for(..., chunk=)` / `(key=)`                                        |
| `ChunkStores.exclusive`, `ClaimService.exclusive`, the `exclusive=` wiring in `composition.py`                                                                                                                                                                                                          | Nothing                                                                         |
| Every `*_locked` twin                                                                                                                                                                                                                                                                                   | The plain method, taking `transaction` ([areas.md](./areas.md))                 |
| The domain helpers named `*_locked`: `effective_locked` (`domain/projects/targets.py:163`), `_claim_locked` (`domain/execution/claim.py:394`), `_group_locked` (`domain/operations/queue.py:802`), `_retire_locked` (`domain/urgency/services.py:285`), `_escalate_locked` (`delivery/hub_node.py:881`) | The same helper without the marker, renamed in the slice that converts its site |

`IChunkExclusiveWrites.locked` already opens `begin()` and one `lock_set` underneath
(`store/internal/chunk_exclusive_store.py:143`), so removing it changes no lock a unit takes. `locked_name` is the
exception: it opens its own scoped write and calls `lock_keys` directly (`graph_store.py:289`), outside the unit's
bookkeeping.

### Handle reads and what replaces them

| Handle read                                | Replaced by                                                                   | Needs `transaction` added                                                    |
| ------------------------------------------ | ----------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| `ILockedChunkRead.facts`                   | `IReadChunkFactsRepository.load_facts` (`ports/facts.py:23`)                  | no                                                                           |
| `ILockedChunkRead.facts_for`               | `IReadChunkFactsRepository.load_facts_for` (`ports/facts.py:32`)              | yes                                                                          |
| `ILockedChunkRead.record`                  | `IReadChunkRecordRepository.get` (`ports/record.py:33`)                       | no                                                                           |
| `ILockedChunkRead.records_for`             | `IReadChunkRecordRepository.get_many` (`ports/record.py:34`)                  | no                                                                           |
| `ILockedChunkRead.route_of`                | `IReadChunkRouteRepository.route_of` (`ports/route.py:21`)                    | no                                                                           |
| `ILockedChunkRead.is_ephemeral`            | `IReadChunkLifecycleRepository.is_ephemeral` (`ports/lifecycle.py:18`)        | yes                                                                          |
| `ILockedChunkRead.standing_edges`          | `list_standing_edges` (`ports/dependencies.py:43`)                            | yes                                                                          |
| `ILockedChunkRead.standing_edges_touching` | `standing_edges_for` (`ports/dependencies.py:60`)                             | no                                                                           |
| `ILockedChunkRead.runner_registration`     | `IReadRunnerRegistry.get_runner` (`domain/runners/registration.py:738`)       | yes                                                                          |
| `ILockedWorkRefRead.live_holders`          | `IReadChunkWorkRefsRepository.live_holders` (`ports/work_refs.py:107`)        | yes                                                                          |
| `ILockedWorkRefRead.source_links`          | `IReadProjectLinks.projects_linking_sources` (`domain/projects/links.py:240`) | yes; the caller maps an unlinked source to `[]`, which the handle did for it |
| `ILockedGraphNameRead.get_enabled_by_name` | `IReadGraphRepository.get_enabled_by_name` (`domain/graph/model.py:1004`)     | yes                                                                          |
| `ILockedGraphNameRead.any_minted`          | `IReadGraphRepository.any_minted` (`domain/graph/model.py:1021`)              | yes                                                                          |

### Store methods that lock outside the declaration

Each runs a store-opened write, or locks mid-unit, so the unit's one declaration does not hold the lock:

| Site                                                                                                                                                      | Lock                                  | Converges in                                                        |
| --------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------- | ------------------------------------------------------------------- |
| `chunk_locked_connection`'s own write (`store/transactions.py:174`): `record_escalation`, `record_question`, `record_hub_artifact`, `record_lease_minted` | the chunk                             | `store-judgments`                                                   |
| `garden_delivery_store.py:77,82`, `review_findings_store.py:82`, `chunk_delivery_store.py:195`                                                            | the chunk; garden delivery's findings | `store-judgments`; the findings in `garden-and-work-item-judgments` |
| `write_landing_rows`, `next_route_seq`, `next_artifact_seq` (`chunk_rows.py:691,412,427`), `_transition_conn`, `_bounce_escalation_conn`                  | the chunk, mid-unit                   | `store-judgments`                                                   |
| `finding_store.py:108` (`moved_findings`)                                                                                                                 | findings                              | `garden-and-work-item-judgments`                                    |
| `flow_store.py:419` (`take_turn`)                                                                                                                         | `flow`                                | `flow-judgments`: the reconciler declares the keys                  |
| `config_apply_store.py:145`                                                                                                                               | the apply's sorted key set            | `config-judgments`: the apply service declares the set              |
| `config_import_store.py:106`                                                                                                                              | forge and secret rows                 | `config-judgments`: the import service declares the keys            |
| `project_link_store.py:74`                                                                                                                                | `project-source-links`                | `converge-record-and-work-refs`                                     |
| `graph_store.py:291`                                                                                                                                      | `graph_name`                          | `converge-graph`, with `locked_name`                                |
| `lock_forge_row`, `lock_secret_row`, the garden-proposal and work-item row locks                                                                          | their rows                            | their judgment slices                                               |

[store-judgments.md](./store-judgments.md) gives each judgment slice's form.

## Flows

### The transitional form

`ILockedChunkRead.transaction` is already a `LockedTransaction` covering the block's chunks and keys. So an area
converts its port methods to the target form while a legacy block still calls them, handing in `handle.transaction`:

```python
with self._exclusive.locked([chunk.chunk_id]) as handle:                      # legacy block, not yet switched
    facts = handle.facts(chunk.chunk_id)
    self._lifecycle.record_stop(chunk.chunk_id, by=by, at=at, transaction=handle.transaction)   # converted
```

A lock site switches to `begin()` in the slice that converts the last method it calls. A block that calls methods of
several areas keeps its legacy opener, passing `handle.transaction`, until every one of them is converted; it then
switches in `converge-cross-area`, and `converge-graph` removes the seam with its last caller. No landing leaves a
method with both forms.

### Reads that miss the unit today

Four legacy blocks read on a separate connection while holding their lock; each joins the unit when its site switches:

- `ApplyService._exit_claimant` reads `route_of` without `transaction` (`domain/execution/apply.py:420,437,604`).
- `ClaimService._claim_locked` reads `self._graphs.get` and, after its write, `load_artifacts` without it
  (`domain/execution/claim.py:414,464`).
- `RestartService.restart` and `EditService.edit` read `self._graphs.is_retired` without it (`restart.py`, `edit.py`).

On SQLite the separate read sees the same state, so joining changes no route's behavior; on Postgres it closes a window
in which the decision reads past its own lock.
