---
name: adapters-stay-apart
depends-on:
  - garden-and-work-item-judgments
  - converge-record-and-work-refs
---

# Adapters stay apart

Hands every write an adapter makes to another repository's facts, through a sibling's private helper, back to the
domain. Delivers `engineering:adapters-stay-apart`. Spec [adapters.md](../spec/adapters.md).

## Change

- `WorkItemStore`'s composite mints become domain compositions in one unit over the work-item, chunk-record,
  run-context, and proposal-closure repositories, declaring the chunk's `WORK_REF` keys beside its urgency keys.
- The routine, routine-scope, review-findings, and garden-delivery adapters' writes to a sibling's tables are moved to
  the domain.
- Config apply, config import, and tenant creation are recorded as declared exceptions, each suppression comment naming
  them so.

## Acceptance

- No adapter but the three declared exceptions calls a sibling adapter's private helper to write that sibling's facts.
- An accept of a proposal already closed writes nothing further and raises `GardenProposalAlreadyClosed`, as today.
- Behavior is unchanged at every route and command (`engineering:behavior-preserving-landings`).
