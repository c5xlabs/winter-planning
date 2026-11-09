# The train's record

Paths are relative to `blizzard/src/blizzard/`.

## Domain

```python
# foundation/ids.py
class IdPrefix(StrEnum):
    ...
    FLOW_BATCH = "fbt"  # a batch a flow proves and lands together


# foundation/hub_event_types.py — flow_facts.kind
class FlowChangeKind(ComponentNamed, StrEnum):
    SUBMITTED = "submitted"
    TURN_STARTED = "turn-started"
    TURN_ENDED = "turn-ended"
    LANDED = "landed"
    WITHDRAWN = "withdrawn"
    BATCH_FORMED = "batch-formed"
    BATCH_BUILT = "batch-built"
    BATCH_LANDED = "batch-landed"
    BATCH_SPLIT = "batch-split"
    BATCH_ABANDONED = "batch-abandoned"
    LEFT_BATCH = "left-batch"
    EJECTED = "ejected"
    TRAIN_HELD = "train-held"
    TRAIN_RELEASED = "train-released"
```

```python
# hub/domain/delivery/flows.py — behind the same ports as the flows epic's types
class BatchState(StrEnum):  # derived from facts and member rows (bzh:facts-not-status), never stored
    BUILDING = "building"
    PROVING = "proving"
    LANDED = "landed"
    SPLIT = "split"
    ABANDONED = "abandoned"


@domain_model
@dataclass(frozen=True)
class BatchMember:
    submission_id: str
    position: int  # arrival order within the batch
    merged_revision: str | None  # the staging branch's tip once this member was merged in


@domain_model
@dataclass(frozen=True)
class Batch:
    batch_id: str
    flow_id: str
    split_from: str | None
    base_revision: str
    staging_ref: str
    train_ref: str
    revision: str | None
    formed_at: datetime
    members: tuple[BatchMember, ...]
    facts: tuple[FlowFact, ...]  # its facts so far
    # ...and the flow's repository and its batching settings, handed to the train step frozen


# Derived, never stored:
# - a flow holds while its newest train-held / train-released fact is train-held;
# - a member's verdict is the newest landed or ejected fact naming its submission.
```

| `flow_facts.outcome` on | Values                                                                                                                       |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `ejected`               | `conflict`, `failure`, `inherited-failure`                                                                                   |
| `batch-abandoned`       | `unproven`, `outpaced`, `advance-refused`, `proof-timed-out`, `member-left` (ejected elsewhere, or withdrawn, while proving) |

| Marker                  | Written by                           | Means                                          |
| ----------------------- | ------------------------------------ | ---------------------------------------------- |
| `train-ready/<repo>`    | The visit, at the submitted revision | Prepared for the flow's train                  |
| `train-fallback/<repo>` | The train                            | This submission lands alone through its policy |
| `queued/<repo>/<n>`     | The visit, after `enqueue`           | Change `n` is in the forge's queue             |
| `merged/<repo>`         | The flow, at the batch's revision    | As today                                       |
| `delivery-findings`     | The flow, on an ejection             | As today                                       |

## Interfaces

### The train turn

```python
# hub/delivery/flow_driver.py — FlowDriver takes a train turn on a free flow when:
#   its batch was just formed, or is due its next proof read (TRAIN_POLL_SECONDS), or, while the flow holds, its next
#   read of the base's checks.
# A train turn goes ahead of any visit's turn on its flow: a batch in progress is the oldest work on its branch.
# A visit's turn on a batching flow is a short read of its verdict and runs between the train's.
# The flows epic's turn guarantees hold unchanged: one open turn per flow; taken and ended in a transaction under the
#   tenant row lock; ended `interrupted` at boot and by the driver's sweep of turns it is not running.
# Each step is a short pass, so a train holds no process or long request and resumes at the step its record reached.
# The step runs on the ActionLoop; the driver writes each fact through to_thread before the step continues and holds
#   no store transaction across the step.
# A train turn ends `pending` while its batch proves, `interrupted` when cut short. An ejection is the verdict that
#   closes its member's submission in flow_submission_closures, as any verdict does.
```

### Awaiting the queue

```python
# hub/domain/graph/actions.py — never authored, refused as a choice at mint by validation.py, never a verdict
RESERVED_WAITS: frozenset[str] = frozenset({"pending", "awaiting-merge", "awaiting-queue"})
```

A pass answers `awaiting-queue` when its chunk's unlanded repositories are `Queued` on a batching flow or in a forge's
queue, or wait behind one ([batches.md](./batches.md#readiness)). It ends its turn with that outcome; its submission
keeps its place on its flows and is ineligible until its next poll is due, after the node's `poll_interval`, as after
`pending`.

`hub_node_poll.kind` gains `awaiting-queue`, written by `HubNodeRouter.record_wait` as `awaiting-merge` is: a run of
them is one row whose `last_polled_at` moves forward in place
([policies](../../../../delivered/advanced-delivery/policies/spec/landing.md#awaiting-a-merge)). It ends a run of
`pending` rows, so the node's `poll_timeout` never turns a chunk back while a train or a forge's queue proves it. Its
bounds are `TRAIN_PROOF_TIMEOUT` on a batch ([batches.md](./batches.md#system)) and `QUEUE_WAIT_TIMEOUT` on a forge's
queue ([queue.md](./queue.md#flows)).

### The landed event

```python
# hub/store/internal/flow_store.py — one transaction, keyed by the flow's repository. engineering:one-proof-per-batch
def record_batch_landed(...):
    for member in batch.members:
        write_marker(member.chunk_id, f"merged/{repo}", batch.revision)  # no visit fence: the branch holds the work
        write_delivery_repo_landed(member.chunk_id, repo, batch.revision)  # whatever has become of the visit since
        write_fact(flow, "landed", member.submission_id, revision=batch.revision)
    write_fact(flow, "batch-landed", batch_id=batch.batch_id)
    write_event("flow-landed", detail)  # one for the advance; no per-member flow-landed
```

```python
# flow-landed detail: the flows epic's fields (FlowLanding.of), plus batch_id
detail = {
    "flow_id": ..., "resource": ..., "repository_id": ..., "branch": ..., "revision": batch.revision,
    "chunk_id": oldest_member.chunk_id,  # the batch's oldest member
    "project_id": oldest_member.project_id,
    "members": [{"chunk_id": ..., "project_id": ...} for each member],
    "batch_id": batch.batch_id,  # absent on a lone landing, whose members has one element
}
```

### Events and escalation causes

| `EventLogKind` (`foundation/event_log.py`) | Severity |
| ------------------------------------------ | -------- |
| `train-unproven`                           | warning  |
| `train-outpaced`                           | warning  |
| `train-advance-refused`                    | critical |
| `train-proof-timed-out`                    | warning  |
| `train-held`                               | warning  |
| `train-released`                           | info     |
| `forge-capability-fallback`                | warning  |

Each joins `EVENT_LOG_SEVERITY` and `tests/support/event_log_kind_census.py`.

```python
# foundation/escalation_causes.py
class EscalationCause(StrEnum):
    ...
    FORGE_QUEUE_REMOVED = "forge-queue-removed"  # a person took the change out, or the queue's checks never ran
    FORGE_QUEUE_STALLED = "forge-queue-stalled"  # still in the queue past QUEUE_WAIT_TIMEOUT
```

### Invariants

`hub/store/internal/invariants.py` gains, beside `hub:one-turn-per-flow`, which holds for train turns as for any:

| Slug                        | Holds                                                                                                                                             |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `hub:one-batch-per-flow`    | No flow has two batches in progress                                                                                                               |
| `hub:batch-members-open`    | Every member of a batch in progress is an open submission on the batch's flow; no chunk withdrawn or ejected is a member of a batch still proving |
| `hub:turn-has-one-subject`  | Every turn names a submission or a batch, never both, never neither                                                                               |
| `hub:flow-fact-has-subject` | Every flow fact names a submission, a batch, or is a hold of its flow                                                                             |

## Flows

### Crash points

Each is a named point in the registry (`bzh:crash-point-registry`), reached by a scenario of the kill-9 sweep
(`tests/crash/test_kill9_sweep.py`) through the test-fenced crash-point mechanism. engineering:train-crash-recovery.

| Point                               | The window                                                                      | What the next pass finds                                                                                                    |
| ----------------------------------- | ------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `train.after-ref.before-record`     | A `create_ref`, `merge_into`, or `delete_ref` answered; its fact is not durable | The branch as the forge holds it: a ref already at its revision, or a merge with nothing left to merge, is recorded as done |
| `train.after-advance.before-landed` | `advance_ref` accepted; the batch's landing record is not durable               | A base holding the batch's revision, which proves the batch landed whatever the record says; it writes what is missing      |
| `train.after-enqueue.before-queued` | The forge took a change into its queue; `queued/<repo>/<n>` is not durable      | `queue_entry` answers the change queued, and the marker is written without a second `enqueue`                               |
| `train.after-split.before-eject`    | `batch-split` is durable; the halves and the ejection they imply are not        | The split re-derived from its fact, forming the half or ejecting the member it names                                        |

## Data

Tenant-owned, written through `ScopedConnections`, beside the flows epic's tables in `hub/store/schema.py`. A batch is
made by the work reaching it, so, like a flow, it has no revision and no change log.

```sql
CREATE TABLE flow_batches (
  batch_id       VARCHAR     NOT NULL,  -- fbt_<ulid>
  tenant_id      VARCHAR     NOT NULL REFERENCES tenants (tenant_id),
  flow_id        VARCHAR     NOT NULL REFERENCES flows (flow_id),
  split_from     VARCHAR     NULL,      -- the batch this one was split from
  base_revision  VARCHAR     NOT NULL,  -- the base's tip the train was built on; changes on a rebuild
  staging_ref    VARCHAR     NOT NULL,  -- <staging_prefix><flow_id>/<batch_id>
  train_ref      VARCHAR     NOT NULL,  -- <train_prefix><flow_id>/<batch_id>
  revision       VARCHAR     NULL,      -- the train's tip once every member is merged
  formed_at      TIMESTAMP   NOT NULL,  -- UtcDateTime
  PRIMARY KEY (batch_id)
);
CREATE INDEX ON flow_batches (tenant_id, flow_id);  -- a flow's batch in progress

CREATE TABLE flow_batch_members (
  batch_id         VARCHAR  NOT NULL REFERENCES flow_batches (batch_id),
  submission_id    VARCHAR  NOT NULL REFERENCES flow_submissions (submission_id),
  tenant_id        VARCHAR  NOT NULL,
  position         INTEGER  NOT NULL,  -- arrival order within the batch
  merged_revision  VARCHAR  NULL,      -- the staging branch's tip once this member was merged in
  PRIMARY KEY (batch_id, submission_id)
);
CREATE INDEX ON flow_batch_members (tenant_id, submission_id);

ALTER TABLE flow_facts ADD COLUMN batch_id VARCHAR NULL;
ALTER TABLE flow_facts ALTER COLUMN submission_id DROP NOT NULL;  -- a batch's facts and a hold are the flow's
ALTER TABLE flow_facts ALTER COLUMN chunk_id DROP NOT NULL;
-- check: a fact names a submission, a batch, or (kind train-held / train-released) neither but its flow

ALTER TABLE hub_turns ADD COLUMN batch_id VARCHAR NULL;
ALTER TABLE hub_turns ALTER COLUMN submission_id DROP NOT NULL;
-- check: exactly one of submission_id, batch_id is set

ALTER TABLE repositories ADD COLUMN batch_size          INTEGER NOT NULL;  -- capabilities.md#the-batching-settings
ALTER TABLE repositories ADD COLUMN batch_wait_seconds  INTEGER NOT NULL;
ALTER TABLE repositories ADD COLUMN train_prefix        VARCHAR NOT NULL;
ALTER TABLE repositories ADD COLUMN staging_prefix      VARCHAR NOT NULL;
```

### Migration

One manual migration (`bzh:manual-migrations`) under `hub/store/migrations/versions/`, chained after the policies
epic's, frozen once released (`bzh:frozen-revisions`), inside the portable surface (`bzh:sql-portable`), proven on
Postgres as on SQLite, every name within Postgres's 63 characters. engineering:one-release, engineering:batching-opt-in.

| Step | Change                                                                                                                                                                                            |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | Add `repositories.batch_size` filled `1`, `batch_wait_seconds` filled `60`, `train_prefix` filled `blizzard-train/`, `staging_prefix` filled `blizzard-staging/`; then tighten each to `NOT NULL` |
| 2    | Create `flow_batches` and `flow_batch_members`, empty, with their indexes                                                                                                                         |
| 3    | Add `flow_facts.batch_id`, null on every row; relax `flow_facts.submission_id` and `chunk_id` to nullable; add the fact-subject check                                                             |
| 4    | Add `hub_turns.batch_id`, null; relax `hub_turns.submission_id` to nullable; add the one-subject check                                                                                            |

No repository changes revision; no `config_changes` row is written.

## Open questions

- The migration's parent: the spec chains it after the policies epic's (`20261029_1200_landing_policies`), but the
  store's head has since moved (`20261106_0900_errand_marks`). Engineering.
- Index and check-constraint names are not given; the schema's convention is `ix_<table>_…` and `ck_<table>_…`.
  Engineering.
- The `Batch` field carrying the flow's repository and batching settings, and its type. Engineering.
- `record_batch_landed`'s signature; the flows epic's `record_landing` takes a fence (`admission`) this write skips.
  Engineering.
