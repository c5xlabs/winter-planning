# Business rules leave the store

How every store write that judges a domain predicate hands its fact back to the domain, one section per slice:
`store-judgments`, `config-judgments`, `garden-and-work-item-judgments`, `flow-judgments`, and `tenant-judgments`.
Delivers `engineering:store-refuses-stale`, ordered before the seam converges by
`decision:transactions/judgments-before-seam`. Paths are under `blizzard/src/blizzard/hub/`, measured on `master` at
`eda353d55`.

A store write may refuse only as stale: a version or monotonic token that moved, a uniqueness constraint, or an
insert-if-absent that found its row (`bzh:store-refuses-stale`). Every refusal type, detail string, and outcome a route
or command returns today is returned unchanged; only who decides it moves.

## Chunks

For `store-judgments`: the four judgments `bzh:store-exclusive-write` lists as Known debt.

### The terminal check in `fence`

`fence` (`store/internal/chunk_rows.py:512`) runs four checks under the chunk's row lock: the newest epoch, then whether
the chunk is terminal, then the epoch's admission, then the claimant. Only the terminal check judges a domain predicate;
the epoch and claimant checks compare tokens and stay.

No token covers terminality. Stop and completion move no epoch, a terminal transition does not raise `latest_epoch`, and
`chunks` carries no revision. So the domain decides terminality inside the unit, from `ChunkFacts` it reads under the
same chunk lock, before calling the write. A chunk-level token that stop, completion, and terminal transitions bump
would admit the compare-and-set form, and is not taken: it needs a revision, and the unit already holds the lock the
decision needs.

```python
# store/internal/chunk_rows.py                                               engineering:store-refuses-stale
def fence(
    conn, chunk_id, *, epoch: int, admission: EpochAdmission, claimant: Claimant | None = None
) -> FenceRefusal | None:
    """Stale epoch, then displaced claimant. ``None`` admits the write. Terminality is the caller's."""

# domain/chunk/ports/fence.py
class FenceRefusal:
    @classmethod
    def terminal(cls, epoch: int) -> "FenceRefusal": ...   # now built by the domain, same detail, same latest=None
```

Every write that runs `fence` today, and the caller that now decides terminality first:

| Store write                                                                      | Domain caller                                                                 | Reads facts under the lock today |
| -------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- | -------------------------------- |
| `ChunkEscalationsStore.record_escalation` (`chunk_escalations_store.py:89`)      | runner `escalation.recorded` (`domain/execution/facts.py:438`); `ApplyService._escalate` (`apply.py:544`) | runner path no; apply yes |
| `ChunkEscalationsStore` bounce (`:184`) and bounce escalation (`:239`)           | `HubNodeRouter._bounce`, `_escalate_locked` (`delivery/hub_node.py:901,885`)  | partly                           |
| `ChunkQuestionsStore.record_question` (`chunk_questions_store.py:91`)            | `domain/execution/questions.py:92`; `facts.py:457`                            | runner path no                   |
| `ChunkDecisionsStore._record_decision_conn` (`chunk_decisions_store.py:320`)     | `DecisionService.submit` (`decisions.py:195`); `gate_arrival.py:30`           | yes                              |
| `ChunkMovementStore._record_transition_conn` (`chunk_movement_store.py:162`)     | `ApplyService._transition` (`apply.py:411`)                                   | yes                              |
| `ChunkMovementStore._record_migration_conn` (`:320`)                             | `apply.py:591`; `hub_node.py:562`                                             | yes                              |
| `ChunkHubExecStore._transition_conn` (`chunk_hub_exec_store.py:107`)             | `hub_node.py:516`                                                             | yes                              |
| `ChunkArtifactsStore.record_hub_artifact` (`chunk_artifacts_store.py:285`)       | `flow_turn.py:465,500,724`; `hub_node.py:489,628,836,976`                     | partly                           |
| `GardenDeliveryStore.deliver` (`garden_delivery_store.py:86`)                    | `domain/garden/delivery/materialize.py:308`                                   | not traced                       |
| `ReviewFindingsStore.deliver` (`review_findings_store.py:85`)                    | `domain/garden/review/materialize.py:189`                                     | not traced                       |
| `write_landing_rows` (`chunk_rows.py:694`), via `record_landing` in the artifacts and flow stores | `flow_turn.py:564,581`                                       | not traced                       |

All eleven move in `store-judgments`, in the same change that takes the terminal check out of `fence`, so no path is
left unchecked between landings. Where the caller does not read facts under the lock today, it gains that read. The
runner's fact path, `_chunk_locked_landing` (`facts.py:426`), is the widest: it calls the store write directly.
`refuse_incoherent_attempt` (`domain/execution/completion.py:103`) checks the current node, open escalation, and open
question but leaves terminality to the fence; it gains the terminal check.

`write_landing_rows` does not refuse a terminal chunk: it writes anyway and returns `refused` for the caller to map to
`EffectStatus.FENCED` (`flow_turn.py:520`). The caller now computes that verdict and passes it, and the store returns
only the stale verdict.

`MintAdmission` (`domain/chunk/ports/fence.py:97`) already holds the rule in the domain, but the store reads its
`terminal` fact and invokes `refusal()` inside `ChunkRouteStore.record_lease_minted` (`chunk_route_store.py:207`). The
domain reads the facts under the lock and calls `refusal()`; the store keeps the epoch and owner check.

### The escalation that runs that fence on its own write

`record_escalation` opens `chunk_locked_connection`, so it either joins the caller's unit or locks its own write, and
runs `fence` with the claimant. It now takes `transaction: LockedTransaction` with no default, so the caller always
holds the chunk lock and has decided terminality. Both callers already open a locked unit (`facts.py:333`,
`apply.py:544`).

### The backlog write that drops a promoted chunk's position

`ChunkQueueStore.record_backlog_positions` (`chunk_queue_store.py:115`) silently drops each pair naming a chunk that is
promoted now. Every caller already decides the backlog's order under the backlog ranking lock, and promote and demote
take both lists' locks, so the filter drops no pair its caller decided on. The filter is deleted, with no replacement.
Its callers are `QueueService.reorder` and `reposition` (`domain/operations/queue.py:650,666`) and the renormalization
in `DemoteService.demote` (`demote.py:100`), all through `write_positions` (`queue.py:561`).

### The promote that writes nothing for an already-promoted chunk

`ChunkQueueStore.record_promote` (`chunk_queue_store.py:68`) returns `None` for a chunk promoted now. It has no
production caller: `PromoteService.promote` (`domain/operations/promote.py:101`) decides the replay itself through
`promotion_writes` (`:47`) and writes through `record_promote_with_position`, as `CadenceMint` does. `record_promote` is
removed from the port and the store; the tests that seed through it move to `record_promote_with_position`.

### Each fenced decision

Each fenced decision runs, inside one locked unit:

1. `begin()`, then `lock_set(chunks=[chunk_id, ...])`.
2. Read `ChunkFacts` joining the unit; refuse as `FenceRefusal.terminal(epoch)` when the chunk is terminal.
3. Call the write, which runs `fence` for the stale epoch and the displaced claimant only.

## Forges and secrets

For `config-judgments`.

A repository or work-source create, edit, or enable refuses a retired forge through `referred_forge`
(`store/internal/forge_record_store.py:90`, raising `ForgeRetired` or `ForgeUnavailable`) and a retired secret through
`_check_secret` (`repository_record_store.py:65`, `work_source_record_store.py:63`, raising
`RepositorySecretUnavailable` or `WorkSourceSecretUnavailable`). Retiring a forge refuses one still in use inside
`record_forge_lifecycle` (`forge_record_store.py:177`, `ForgeInUse`), and retiring a secret one still referenced inside
its `record_lifecycle_fact` (`secret_store.py:94`). The rules are already domain functions, `require_unused`
(`domain/config/forges.py:255`) and `require_unreferenced` (`domain/config/secrets.py:162`); the store calls them over
facts it reads. Each side locks the forge or secret row with a no-op `UPDATE` it runs for itself (`lock_forge_row`,
`lock_secret_row`).

The row locks become declared keys, `LockNamespace.FORGE` and `LockNamespace.SECRET`, keyed by the record's id:

1. A referrer's write opens `begin()` and declares the forge and secret keys it names.
2. It reads the forge's and the secret's retirement joining the unit, and the domain refuses with the type it raises
   today.
3. A retirement declares its record's key, reads its active referrers joining the unit, and runs `require_unused` or
   `require_unreferenced` in the domain before writing the lifecycle fact.

A referrer and a retirement of the same forge or secret take the same key, so neither decides past the other. Config
apply and import write the same referrers and declare the same keys ([areas.md](./areas.md) §Open questions).

## Garden proposals, work items, and urgency moves

For `garden-and-work-item-judgments`.

**Garden proposals.** `GardenProposalStore.edit`, `attach`, and `detach` (`garden_proposal_store.py:169`) write nothing
to a closed proposal (`is_closed`, `:152`); their callers are `domain/garden/proposals/model.py:571,579,587`. A closure
locks the proposal row with a no-op `UPDATE` inside `insert_garden_proposal_closure_row`. Both sides declare
`LockNamespace.GARDEN_PROPOSAL`, keyed by the proposal's id; the edit re-reads the proposal's closure joining the unit,
and the domain returns the same no-op outcome for a closed one.

**Work items.** `WorkItemEditService.edit` (`domain/work_items/editing.py:250`) runs `require_open_for` on an item read
before any lock, and `WorkItemStore.edit` (`work_item_store.py:258`) guards the race with `closed_at IS NULL`. Edit,
close, and `withdraw_open` declare `LockNamespace.WORK_ITEM`, keyed by the item's source and ref; the edit re-reads the
item joining the unit and runs `require_open_for` there, raising `WorkItemNotEditable` as today, and the store's edit
writes unconditionally.

**Urgency moves.** Retiring an urgency moves its open holders to another (`domain/urgency/services.py:322`), and
`UrgencyChunksStore.move` (`urgency_chunks_store.py:82`) skips a terminal chunk through `maybe_live`, so it keeps its
urgency (the contract at `domain/urgency/ports.py:185`). The retirement reads which holders are terminal joining the
unit and names the chunks to move; the store moves only those. The retirement locks the urgency keys, not its holders'
chunks, so a holder stopped between the read and the move is moved: an accepted exception to the contract
(`decision:transactions/urgency-move-race`), which the contract's docstring states.

## Flow submissions and turns

For `flow-judgments`.

`FlowStore.close` (`flow_store.py:400`) writes nothing for a submission with a turn open, and `FlowStore.take_turn`
(`:406`) writes nothing for a submission that is closed, has a turn open, is busy (`_busy`, `:647`), or has another
ahead of it (`_ahead_on`, `:659`); `take_turn` also locks the `flow` key for itself (`:419`). Both are called by the
flow-line reconciler (`domain/delivery/reconcile.py:265,440`).

**Proposed.** The reconciler opens `begin()` declaring the submission's chunk and its `LockNamespace.FLOW` key, reads
the submission, its turns, and the queue ahead joining the unit, and decides; the store's `close` and `take_turn` take
`transaction: LockedTransaction` and write unconditionally.

`flow_rows.withdraw_waiting` (`flow_rows.py:175`) runs inside a stop, completion, or restart write
(`chunk_lifecycle_store.py:65,86`, `chunk_movement_store.py:426`) and picks the chunk's open submissions with no turn
open. The lever's domain service reads them joining its unit, under the chunk lock, and names the ones to withdraw;
`withdraw_waiting` withdraws the named ones. Because `take_turn` also declares the chunk, a lever and a turn on the same
chunk exclude each other on the chunk lock.

## Tenant rename and close

For `tenant-judgments`.

`TenantAdministrationStore.rename` (`tenant_admin_store.py:134`) writes nothing for a tenant being closed
(`~_DELETING`); its caller is `domain/tenancy/administration.py:190`. `TenantTeardownStore.close`
(`tenant_teardown_store.py:81`) locks every tenant row in `tenant_id` order (`_lock_every_tenant`, `:45`) and refuses
`LastTenant` when no other tenant is open; its caller is `domain/tenancy/teardown.py:179`.

Both move onto the tenant's ordinary unit. Its opening lock already refuses a closing tenant, which is rename's whole
check, and close declares the hub-wide `TENANT_LIFECYCLE` lock in place of locking every tenant row, then refuses
`LastTenant` in the domain. [hub-locks.md](./hub-locks.md) owns the table, the lock order, and both flows.

