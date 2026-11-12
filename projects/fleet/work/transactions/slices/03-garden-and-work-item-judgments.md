---
name: garden-and-work-item-judgments
---

# Garden and work-item judgments

Hands the closed-proposal, closed-work-item, and urgency-move rules the store judges back to the domain. Delivers
`engineering:store-refuses-stale` for garden proposals, work items, and urgency moves. Spec
[store-judgments.md](../spec/store-judgments.md#garden-proposals-work-items-and-urgency-moves).

## Change

- A garden proposal's edit, attach, and detach, and a work item's edit, lock the record they change, re-read it under
  the lock, and refuse a closed one in the domain; the closure of each takes the same lock.
- An urgency retirement locks the chunks it moves, reads which are terminal, and moves only the rest.
- The store's `is_closed`, `closed_at IS NULL`, and `maybe_live` conditions leave those writes.

## Acceptance

- None of these writes reads a closure or terminal fact to refuse or to skip.
- An edit racing a closure is refused exactly as today, and a chunk already terminal when the retirement reads it keeps
  its urgency (`decision:transactions/urgency-move-race`).
- Behavior is unchanged at every route and command (`engineering:behavior-preserving-landings`).
