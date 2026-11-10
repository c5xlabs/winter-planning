---
epic: steering
refinement: outline
status: planned
---

# Steering a running chunk

A standalone epic; its requirements are inline below.

## The problem

The operator watches a chunk head somewhere wrong and has two options, both bad: watch the night go to waste, or take
the session over by hand.

## The outcome

The operator does what they would do with a colleague: says one sentence and lets the agent carry on.

## What this epic is not

It builds a word to a chunk already running and nothing more: not a conversation, not a takeover. The narrow shape is
the point, because an open channel to a working agent is a different product, and the human-channel taxonomy stays small
enough to hold in the head.

## Requirements

### One message to a running chunk (product:steer-message)

The operator writes a sentence; the agent receives it and continues.

Acceptance:

- The operator sends one message to a running chunk, and the agent continues with it in hand.

### Delivered at the next natural pause (product:pause-delivery)

The word arrives where the agent can act on it rather than mid-thought (`decision:steering/finished-chunk-message`,
`decision:steering/second-message`).

Acceptance:

- A message sent while the agent is mid-step reaches it when that step ends, not partway through.
- The agent's next step after the pause takes the message into account.

### Recorded in the chunk's history (product:steer-recorded)

A human entry into a running chunk is part of its history like every other human entry, so the morning's account of what
happened includes what was said to it.

Acceptance:

- The chunk's history shows each steering message sent to it.

### Reachable where the chunk is watched (product:steer-on-chunk)

The affordance lives on the chunk as the operator is looking at it, not on a separate screen they must go and find.

Acceptance:

- The operator steers a chunk from the chunk's own view.
