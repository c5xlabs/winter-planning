# Trains engineering requirements

Engineering requirements of `epic:advanced-delivery/trains`.

## What already holds

This epic teaches a flow to land many submissions at once, and to hand its line to a forge that keeps one of its own. It
builds on the delivered `epic:advanced-delivery/actions`, `epic:advanced-delivery/forges`,
`epic:advanced-delivery/flows`, and `epic:advanced-delivery/policies`, and on `epic:advanced-delivery/gitlab`:

- An action is pure under `bzh:hub-action-purity`, receiving a frozen input and handing its effects to an `IEffectSink`
  the driver persists; `land` runs each repository's policy a pass at a time
  ([actions spec](../../../delivered/advanced-delivery/actions/spec/index.md)).
- The forge roles, the registry that hands them out, and the capabilities are the forges epic's
  ([forges spec](../../../delivered/advanced-delivery/forges/spec/index.md)). Every forge is spoken through the async
  roles `ForgeRegistry` hands out: `IForgeRefs`, including `merge_into` and the compare-and-swap `advance_ref`,
  `IForgeChanges`, and `IForgeChecks`.
- The flow, its submissions, its turns, its record, and how a verdict reaches a chunk are the flows epic's
  ([flows spec](../../../delivered/advanced-delivery/flows/spec/index.md)). Each branch's flow gives its submissions
  turns, one at a time, in arrival order.
- The repository's policy, the order a project lands its repositories in, the reserved waits, and the rate budget are
  the policies epic's ([policies spec](../../../delivered/advanced-delivery/policies/spec/index.md)).
- The GitLab adapter, its vocabulary, and the mock's GitLab face are the gitlab epic's
  ([adapter](../gitlab/spec/adapter.md), [mock](../gitlab/spec/mock.md)). GitHub and GitLab both sit behind the same
  roles, so a GitLab project's merge train is reached through the same seam as GitHub's merge queue.

## Brief requirements

- engineering:one-release — The epic ships in one release, with one manual migration
  ([record.md](./spec/record.md#migration)).
- engineering:mock-proven — Every acceptance bullet in this file is proven against the mock forge
  ([mock.md](./spec/mock.md)).

## Batching is opt-in (engineering:batching-opt-in)

An installation that predates this epic lands exactly as it did: every repository's `batch_size` is one, so every flow
lands its submissions alone, and no repository's policy changes. Batching and the forge's queue are an operator's
choice, repository by repository. Serves `product:batch-settings`.

Acceptance:

- A fixture store captured before the migration carries over and lands every chunk alone, with the outcomes and markers
  it would have before.

## One proof per batch (engineering:one-proof-per-batch)

A batch lands with one advance of the base to its proven revision, and its checks run once, on the train branch's
finished tree. Serves `product:batch-proof` and `product:proven-tree-lands`.

Acceptance:

- Four chunks ready on one branch land in one advance; the base's revision is the batch's revision; each chunk's
  `merged/<repo>` names it; each change reads merged; one `flow-landed` carries all four in `members`.
- The train branch is pushed once, at the batch's finished tree; no check runs on the staging branch.
- One chunk that holds its base and is green on its own revision lands by an advance to that revision, without a train
  or a second run of checks.
- A batch proven and then outpaced by a direct push to the base is rebuilt and proven again before the base advances;
  the base never holds a tree that was not proven.

## Mechanical joins (engineering:mechanical-joins)

A member is never brought up to its base on its own branch; the train merges it onto the base. Serves
`product:mechanical-first`.

Acceptance:

- A chunk behind its base joins a batch with no update to its own branch and no agent woken.
- One that conflicts only with a chunk ahead of it in the batch lands in the next batch or is ejected `conflict` there.
- One that conflicts with the base is ejected `conflict` at once.
- A chunk whose own checks have already failed is answered `failure` and never joins.

## Split bound (engineering:split-bound)

A failing batch is split until the member responsible is found
(`decision:advanced-delivery/trains/split-by-arrival-order`). Serves `product:batch-proof` and `product:ejection`.

Acceptance:

- A batch of eight with one bad chunk lands the seven and ejects the one `failure`, with findings, in no more than seven
  proofs; the seven land in arrival order.
- A check that fails once and passes on its re-run ejects no one.

## Holding and bounding a train (engineering:train-bounds)

A train waits on a red base rather than ejecting a member per batch, and a proof is bounded by `TRAIN_PROOF_TIMEOUT`
rather than the node's `poll_timeout`. Serves `product:train-stops`.

Acceptance:

- A batch whose only failures the base shares ejects its oldest member `inherited-failure`, holds the flow, forms no
  further batch while the base is red, and resumes once the base's checks pass; no second member is ejected for the same
  red base.
- A split that runs past the node's `poll_timeout` turns no member back, since its members answer `awaiting-queue`.
- A check left queued forever is re-run once at `TRAIN_PROOF_TIMEOUT`, then the batch is abandoned `proof-timed-out` and
  formed afresh.

## Order across trains (engineering:train-order)

A chunk's batching repositories land in its project's landing order, and a member that leaves one batch leaves every
batch still proving. Serves `product:batch-proof` and `product:ejection`.

Acceptance:

- A chunk reaching two batching repositories in one project lands them one after the other, in that project's landing
  order; a chunk reaching a batching and a non-batching repository lands each in order; two projects ordering the same
  pair oppositely each land by their own order.
- A chunk ejected from one repository's batch while it sits in another's batch, still proving, abandons that batch,
  which is rebuilt and lands without it.
- A member stopped while its batch proves abandons the batch, which lands without it; a member stopped after the advance
  is recorded landed.

## Abandoned trains and probes (engineering:train-fallback)

A train the repository's CI or protection cannot carry is abandoned and its members land alone, and a probe the forge
cannot answer leaves the last answer standing. Serves `product:probed-capabilities`.

Acceptance:

- A repository whose CI ignores its train branches abandons the batch `unproven` and lands each member alone.
- A protected base abandons the batch `advance-refused`, remembers the refusal, and lands each member alone, and no
  later scheduled probe offers `refs.advance` again until one is taken on demand.
- A probe answered by a 5xx leaves the last answer standing; a repository never probed answers `pending` until a probe
  succeeds; a base requiring linear history cannot batch.

## Crash recovery (engineering:train-crash-recovery)

Each of the train's crash points ([record.md](./spec/record.md#crash-points)) recovers on the next pass without
repeating a forge write, and the train's invariants hold.

Acceptance:

- Each crash point, reached by the kill-9 sweep, recovers on the next pass: a hub killed after the advance and before
  the record writes the record, merging nothing twice; one killed after an enqueue writes `queued/<repo>/<n>` without
  enqueuing again.
- `hub:one-batch-per-flow`, `hub:batch-members-open`, `hub:turn-has-one-subject`, `hub:flow-fact-has-subject`, and the
  flows epic's invariants pass on the carried-over store and after every scenario in this file.

## Forge-queue verdicts (engineering:forge-queue-verdicts)

A `forge-queue` repository's outcomes in the forge's queue map to verdicts, escalations, or a re-enqueue
([queue.md](./spec/queue.md)). Serves `product:forge-queue`.

Acceptance:

- A `forge-queue` repository lands through the mock's merge queue on its GitHub face and its merge train on its GitLab
  face.
- A conflict and failed checks reach the chunk as its verdict.
- A person's removal and a timed-out queue escalate to a person; a cleared queue re-enqueues; a stopped chunk's change
  is dequeued.
- With the queue switched off, the repository lands by `merge-on-checks`, unbatched, and records
  `forge-capability-fallback`.

## Trains documentation (engineering:trains-docs)

The epic reaches beyond `blizzard`'s source. Serves `epic:advanced-delivery/trains`.

- In `blizzard-context`, `domain/artifacts/delivery.md` says a branch's flow may land a batch, that on a batching
  repository the revision landed is the revision proven, and that a chunk is ejected rather than rebased by an agent
  when it cannot join; `domain/operations.md` gains the event kinds in [record.md](./spec/record.md);
  `architecture/crash-correctness/hub.md` gains the train's crash points;
  `verification/blizzard/e2e-scenarios/delivery-policy.md` gains the batching and queue scenarios, and
  `verification/blizzard/e2e-scenarios/board.md` the batch on a lane.
- `docs/deployment/` gains a page on batching, covering the CI a repository must run on its train branches and must not
  run on its staging branches, and that the repository chooses which checks those are, and on handing the line to a
  forge's queue, including the `merge_group` trigger GitHub's merge queue needs.
