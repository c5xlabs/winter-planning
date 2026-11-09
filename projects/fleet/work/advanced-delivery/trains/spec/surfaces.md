# Surfaces

Paths are relative to `blizzard/`. Every surface reads a batch; none forms, splits, abandons, or reorders one. The
batching settings and `forge-queue` travel the repository's doors (API, CLI, admin form, configuration document) as the
policies epic's fields do.

## Interfaces

### Flows API

```python
# src/blizzard/wire/flow.py — product:train-on-board
class BatchMemberView(WireModel):
    chunk_id: str
    position: int
    merged: bool  # merged onto the train yet


class BatchView(WireModel):
    batch_id: str
    state: BatchState  # building | proving | landed | split | abandoned
    train_ref: str
    train_url: str  # the train branch's web address
    base_revision: str
    revision: str | None
    state_since: str  # how long it has been in its state
    split_from: str | None
    members: list[BatchMemberView]


class FlowHoldView(WireModel):
    since: str  # when the hold began
    base_revision: str  # the base revision whose checks are red


class FlowView(WireModel):
    ...
    batch: BatchView | None = None  # the batch in progress
    last_batch: BatchView | None = None  # the newest finished batch, with how it ended
    hold: FlowHoldView | None = None  # set while the flow holds for a red base


class FlowFactView(WireModel):
    ...
    batch_id: str | None = None  # /api/flows/{flow_id}/record reads a batch's whole history in order


# src/blizzard/foundation/flow_place_state.py — FlowPlaceView.state and ChunkFlowPlaceView.state
class FlowPlaceState(StrEnum):
    ...  # landing, next, waiting, polling, awaiting-merge, passed-over
    READY = "ready"  # prepared, waiting for a batch
    IN_BATCH = "in-batch"  # carried by the batch in progress
    HELD = "held"  # prepared on a flow that holds
    IN_FORGE_QUEUE = "in-forge-queue"  # forge-queue repository; carries the forge's position
```

### Repositories API

| Route                                            | Change                                                                                                                                                                                                                                                                                                                                         |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `POST /api/repositories`, `PATCH …/{repository}` | Request gains `batch_size`, `batch_wait_seconds`, `train_prefix`, `staging_prefix`; `landing_policy` admits `forge-queue`. `422 batching_not_applicable`, `422 forge_capability_missing` ([capabilities.md](./capabilities.md#the-batching-settings)). A write that falls back answers `200`/`201` with a `warnings` entry naming the fallback |
| `GET /api/repositories/{repository}`             | `RepositorySummary` gains the four settings and the last probe: the probed capabilities it showed, when it was taken, `advance_refused_at`                                                                                                                                                                                                     |

The schema lists each policy's fallback. `EscalationCause` gains `forge-queue-removed` and `forge-queue-stalled` in the
OpenAPI enum. `openapi/hub.openapi.json` is re-exported and the web client regenerated.

### Live channel

```python
# src/blizzard/wire/sse.py — published for every batch fact and every hold, as for a submission's
class FlowChangedPayload(SseFramePayload):
    flow_id: str
    kind: FlowChangeKind  # gains the batch and hold kinds (record.md#domain)
    submission_id: str
    chunk_id: str
    outcome: str | None = None
    batch_id: str | None = None
```

`contracts/sse/` and the invalidation row in `web/projects/fleet/src/lib/sse/fleet-live.ts` learn `batch_id`.

### CLI

| Command                                | Change                                                                                       |
| -------------------------------------- | -------------------------------------------------------------------------------------------- |
| `blizzard hub flow show`               | Prints the batch in progress, or the hold, under the line; `--record` prints batch facts     |
| `blizzard hub repo create`, `edit`     | `--batch-size`, `--batch-wait`, `--train-prefix`, `--staging-prefix`; `--policy forge-queue` |
| `blizzard hub repo probe <repository>` | New: takes and prints a probe; clears a remembered refused advance                           |

`contracts/cli/hub.json` is regenerated.

### Admin screens

| Screen            | Shows                                                                                                                                                                                                                                                                                                                                                                    |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Repository form   | The batching settings when the policy is `fast-forward` or `merge-on-checks`, noting CI must run on pushes under the train prefix and not the staging prefix, and that a base requiring linear history or signed commits cannot batch. `forge-queue`, with its fallback named when the last probe showed no queue, noting the `merge_group` trigger GitHub's queue needs |
| Repository detail | The last probe, any remembered refused advance, and a button that takes a new probe                                                                                                                                                                                                                                                                                      |

### The board

The flow lanes are in `web/projects/hub/src/app/delivery/`.

| Lane state               | Shows                                                                                                                                                |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| Batch in progress        | Where a lone landing shows its chunk: a group of the chunks it carries, titled with state and time in it ("proving 4 · 6m"), the train branch linked |
| Ready for the next batch | In the line behind the batch                                                                                                                         |
| Held                     | At the head of the lane: "held · base red since 14:02", the base's failing revision linked                                                           |
| Ejected in the last day  | At the lane's end with its verdict and the batch it left, until the board window passes                                                              |
| Split                    | The half being proven, labelled as split from the batch before                                                                                       |
| `forge-queue`            | Each chunk's place in the forge's queue beside the order the flow received them; the forge's queue linked                                            |

| Card's line under its node                         | When             |
| -------------------------------------------------- | ---------------- |
| "in a batch of 4 on `blizzard/master`"             | `in-batch`       |
| "ready for the next batch on `blizzard/master`"    | `ready`          |
| "held for a red base on `blizzard/master`"         | `held`           |
| "3rd in GitHub's merge queue on `blizzard/master`" | `in-forge-queue` |

The chunk detail's timeline shows each batch its submission joined, how each ended, and its ejection, linked to the
flow's record. `CAUSE_SENTENCES` (`web/projects/fleet/src/lib/chunk/chunk-detail/chunk-escalation.ts`) gains a sentence
for each new cause.

## Open questions

- Every wire field name above other than `batch`, `last_batch`, `hold`, and `batch_id` is a placeholder; the spec names
  their contents, not their names. Engineering.
- The field `in-forge-queue` places carry the forge's position in. Engineering.
- The shape of the fallback warning: `RepositorySummary.warnings` today is a deprecated list of `OldFieldWarning`
  (`field`, `forge_id`, `forge_slug`, `message`). Engineering.
- `FlowChangedPayload` and `FlowFactView` require `submission_id` and `chunk_id`; a batch fact or a hold has neither,
  and the spec does not say how the wire carries that. Engineering.
- The shape of the probe on `RepositorySummary` ([capabilities.md](./capabilities.md#open-questions)). Engineering.
