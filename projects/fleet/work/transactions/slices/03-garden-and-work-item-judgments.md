---
name: garden-and-work-item-judgments
external: github:paul-gross/blizzard#2150
depends-on:
  - store-judgments
  - config-judgments
---

# Garden and work-item judgments

Hands the closed-proposal, closed-work-item, and urgency-move rules the store judges back to the domain. Delivers
`engineering:store-refuses-stale` for garden proposals, work items, and urgency moves. Spec
[store-judgments.md](../spec/store-judgments.md#garden-proposals-work-items-and-urgency-moves).

## Change

- A garden proposal's edit, attach, and detach, and a work item's edit, declare the record's `GARDEN_PROPOSAL` or
  `WORK_ITEM` key, re-read it under the lock, and answer a closed one in the domain; the closure of each, and a work
  item's `withdraw_open`, declare the same key.
- A chunk delete pre-reads its `hub:` refs, declares their `WORK_ITEM` keys, and runs again when the refs it re-reads
  under the lock lie outside them.
- Garden delivery and the finding verbs declare `FINDING` keys in place of their row locks.
- An urgency retirement reads which of the holders it already locks are terminal, and names the chunks to move.
- The store's `is_closed`, `closed_at IS NULL`, and `maybe_live` conditions leave those writes.

## Acceptance

- None of these writes reads a closure or terminal fact to refuse or to skip.
- Every proposal edit, attach, detach, and closure, and every work-item edit, close, and withdrawal, declares the
  record's key; a closed proposal's edit is a no-op and a closed work item's edit is refused `WorkItemNotEditable`, as
  today.
- A retirement moves no holder that is terminal under its lock (`decision:transactions/urgency-move-race`).
- Behavior is unchanged at every route and command (`engineering:behavior-preserving-landings`).
