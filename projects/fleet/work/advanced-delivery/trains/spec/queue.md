# Handing the line to the forge

Paths are relative to `blizzard/src/blizzard/`.

## System

| Forge  | Queue                 | Proves and merges                                   |
| ------ | --------------------- | --------------------------------------------------- |
| GitHub | Merge queue           | The merge group it proved, on its `merge_group` run |
| GitLab | Merge train (Premium) | The merged result its pipeline ran on               |

The forge orders the line and owns the promise that what it merges is what it tested. The flow still records every
submission, turn, and verdict on the branch. A repository's CI must listen for GitHub's `merge_group` event; without it
every change leaves the queue `timed-out`.

## Domain

```python
# foundation/landing_policy.py
class LandingPolicy(ComponentNamed, StrEnum):
    MERGE_ON_CHECKS = "merge-on-checks"
    FAST_FORWARD = "fast-forward"
    PARK = "park"
    FORGE_QUEUE = "forge-queue"  # product:forge-queue


# hub/domain/config/landing_policies.py — decision:advanced-delivery/trains/forge-queue-fallback
POLICY_CAPABILITIES[LandingPolicy.FORGE_QUEUE] = PolicyCapabilities(
    requires=frozenset({REFS_READ, COMPARE, CHANGES_OPEN, CHANGES_READ, CHECKS_READ, QUEUE_JOIN}),
)
# The one policy with a fallback: merge-on-checks, unbatched (batch_size > 1 is refused beside forge-queue).
```

```python
# hub/domain/actions/requirements.py — the fallback is applied before any capability is checked.
# land's entry in ACTION_REQUIREMENTS answers POLICY_CAPABILITIES for the policy that will actually run: a
# forge-queue repository whose last probe shows no queue.join requires what merge-on-checks requires, so the forges
# epic's check before a pass never refuses it for the queue it is about to do without.
```

```python
# hub/forge/landing.py
class QueueState(StrEnum):
    QUEUED = "queued"
    PROVING = "proving"
    MERGED = "merged"
    REMOVED = "removed"


class QueueRemoval(StrEnum):
    """Why a change left the queue without merging."""

    CONFLICT = "conflict"
    CHECKS_FAILED = "checks-failed"
    TIMED_OUT = "timed-out"
    MANUAL = "manual"
    CLEARED = "cleared"  # taken out by something other than the change: the queue emptied or rolled back
    OTHER = "other"  # the forge's own words in `detail`


@domain_model
@dataclass(frozen=True)
class QueueEntry:
    state: QueueState
    position: int | None
    removal: QueueRemoval | None
    detail: str
    landed_revision: str | None  # the revision the forge merged; the same merged_revision IForgeChanges reads


class IForgeQueue(Protocol):
    """A role beside IForgeRefs, IForgeChanges, IForgeChecks; ForgeRegistry hands it out only for a repository whose
    probe shows queue.join. Async, through the forges epic's retry policy."""

    async def enqueue(self, number: int, expected_head: str) -> QueueEntry:
        """ForgeRejected, with the forge's reason, when the forge will not take the change."""

    async def queue_entry(self, number: int) -> QueueEntry | None:
        """None when the change was never queued."""

    async def dequeue(self, number: int) -> bool:
        """Whether the forge took the change out."""


@dataclass(frozen=True)
class RepositoryForge:
    ...
    queue: IForgeQueue | None = None  # set only for a repository whose probe shows queue.join
```

## Interfaces

### GitHub

| Operation     | Call                                                                                                       |
| ------------- | ---------------------------------------------------------------------------------------------------------- |
| `enqueue`     | GraphQL `enqueuePullRequest` with `expectedHeadOid`, keyed by the pull request's node id                   |
| `queue_entry` | The pull request's `mergeQueueEntry` (state, position); once gone, its newest `RemovedFromMergeQueueEvent` |
| `dequeue`     | GraphQL `dequeuePullRequest`                                                                               |

| `RemovedFromMergeQueueEvent` reason      | `QueueEntry`               |
| ---------------------------------------- | -------------------------- |
| `MERGE_CONFLICT`                         | `removed`, `conflict`      |
| `CI_FAILURE`                             | `removed`, `checks-failed` |
| `CI_TIMEOUT`                             | `removed`, `timed-out`     |
| `MANUAL`                                 | `removed`, `manual`        |
| `QUEUE_CLEARED`, `ROLL_BACK`             | `removed`, `cleared`       |
| `ALREADY_MERGED`, or merged by the queue | `merged`, never a removal  |
| Anything else                            | `removed`, `other`         |

### GitLab

| Operation     | Call                                                                              |
| ------------- | --------------------------------------------------------------------------------- |
| `enqueue`     | `POST /projects/:id/merge_trains/merge_requests/:iid` with `sha` and `auto_merge` |
| `queue_entry` | `GET /projects/:id/merge_trains/merge_requests/:iid`                              |
| `dequeue`     | Cancel the merge request's auto-merge, which takes it off its train               |

| Car status                    | `QueueState` |
| ----------------------------- | ------------ |
| `idle`                        | `queued`     |
| `fresh`, `stale`, `merging`   | `proving`    |
| `merged`, `skip_merged`       | `merged`     |
| Gone from the train, unmerged | `removed`    |

GitLab says why a change left only in a system note: the adapter reads the merge request's newest system note about the
train and maps the phrasings for a failed pipeline (`checks-failed`), a conflict (`conflict`), and a person's removal
(`manual`); any other is `other`, with the note in `detail`.

## Flows

```python
# hub/delivery/actions/landing/forge_queue.py — per repository, per pass. engineering:forge-queue-verdicts
async def step(repo: RepoLanding) -> RepoState:
    change = find_or_open_change(); write("delivery-pr/<repo>/<n>"); gate_head()  # as merge-on-checks does
    if not queued:
        verdict = read_own_checks()  # shared reading; the queue takes only a change passing them
        if verdict.running:
            return Waiting(...)
        if verdict.failed:
            return Failure(...) or Inherited(...)  # as for merge-on-checks
        # Ready.land is enqueue; not taken until every repository before it in its project's landing order landed.
        return Ready(land=enqueue_then_write("queued/<repo>/<n>"))
    entry = await repo.forge.queue.queue_entry(n)
    match entry.state, entry.removal:
        case "queued" | "proving", _:
            return Queued()  # the visit answers awaiting-queue (record.md#awaiting-the-queue)
        case "merged", _:
            return Landed(entry.landed_revision)
        case "removed", "conflict":
            return Conflict(...)
        case "removed", "checks-failed":
            return Failure(findings=queue_failing_checks)
        case "removed", "cleared":
            # a fresh enqueue on the next pass, at most twice per visit; then failure naming the reason
            ...
        case "removed", "manual" | "timed-out":
            # escalated as the policies epic escalates a parked change a person closed unmerged
            return NeedsPerson(ActionEscalation(EscalationCause.FORGE_QUEUE_REMOVED, change_and_reason))
        case "removed", "other":
            return Failure(findings=entry.detail)
    # Past QUEUE_WAIT_TIMEOUT (3 h from the enqueue):
    #   NeedsPerson(EscalationCause.FORGE_QUEUE_STALLED, change and its last place in the queue)
```

```python
# Without queue.join at a visit (the queue switched off since): land by merge-on-checks, unbatched, and record one
# forge-capability-fallback event (warning) per visit naming the repository and the capability.
# Setting forge-queue on a repository whose probe shows no queue.join: accepted, answered with a warning naming the
# fallback (surfaces.md#api). Every other missing capability is refused when set and escalated at a visit.
```

While its change sits in the queue, a visit's turns are short reads. Its submission keeps its place without holding the
flow's next turn from anyone. Its wait is bounded by `QUEUE_WAIT_TIMEOUT`, not the node's poll timeout.

```python
# A lever that withdraws a queued submission — stop, restart, complete:
await repo.forge.queue.dequeue(n)
# A dequeue the forge refuses, or a change merged before it is dequeued: recorded as landed on the flow, as a pass
# already running is under the flows epic.
```

## Open questions

- `QueueEntry.position`'s type, and whether it is set in every state; the spec names it only as a place in the queue.
  Engineering.
- The GitLab system-note phrasings the adapter maps are not listed; they come from the recorded Premium fixtures.
  Engineering.
- Where `QUEUE_WAIT_TIMEOUT` is declared. Engineering.
