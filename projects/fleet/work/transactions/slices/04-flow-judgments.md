---
name: flow-judgments
---

# Flow judgments

Hands the flow-queue rules the store judges back to the domain. Delivers `engineering:store-refuses-stale` for flow
submissions and turns. Spec [store-judgments.md](../spec/store-judgments.md#flow-submissions-and-turns).

## Change

- Closing a submission and taking a turn open one unit declaring the flow key, read the submission's turns and the
  queue ahead of it joining the unit, and decide in the domain.
- A stop, completion, or restart reads its chunk's waiting submissions under the chunk lock and names the ones to
  withdraw; `withdraw_waiting` writes the named ones only.
- `FlowStore.take_turn` stops locking the flow key for itself.

## Acceptance

- No flow write reads an open-turn, busy, ahead, or waiting fact to refuse or to choose.
- A close with a turn open, and a turn the queue does not allow, write nothing, as today.
- Behavior is unchanged at every route and command (`engineering:behavior-preserving-landings`).
