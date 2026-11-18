---
name: flow-judgments
external: github:paul-gross/blizzard#2151
depends-on:
  - store-judgments
  - config-judgments
---

# Flow judgments

Hands the flow-queue rules the store judges back to the domain. Delivers `engineering:store-refuses-stale` for flow
submissions and turns. Spec [store-judgments.md](../spec/store-judgments.md#flow-submissions-and-turns).

## Change

- Closing a submission and taking a turn pre-read the submission's flows, open one unit declaring its chunk and a `FLOW`
  key per flow, re-read the flows, turns, and queue ahead joining the unit, and decide in the domain; a flow outside the
  declared keys runs the unit again.
- `FlowStore.take_turn` stops locking the flow key for itself, and the `FLOW` key builder moves to the domain.
- A stop, completion, or restart reads its chunk's waiting submissions under the chunk lock and names the ones to
  withdraw; `withdraw_waiting` writes the named ones only.

## Acceptance

- No flow write reads an open-turn, busy, ahead, or waiting fact to refuse or to choose.
- Every close and turn declares the submission's chunk and its flows' keys.
- A close with a turn open, and a turn the queue does not allow, write nothing, as today.
- Behavior is unchanged at every route and command (`engineering:behavior-preserving-landings`).
