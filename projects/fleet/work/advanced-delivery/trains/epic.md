---
epic: advanced-delivery/trains
refinement: refined
status: planned
---

# Trains

Part of `initiative:advanced-delivery`. Its product requirements are in `requirements.md`.

## The problem

When many agents finish at once, landing them one by one is a slow way to discover that their work was fine. Each
landing moves the branch under every chunk still waiting. The forge brings each of those chunks up to the new base
without an agent, but every update throws away a finished run of checks and starts another, so on a busy night the line
on a branch grows faster than it drains. Most of what the fleet spends on it is re-running checks it has already passed,
and a chunk that waits through enough of those runs is turned back to an agent by its node's poll timeout, though its
work was right all along.

## The outcome

The flow on a branch becomes a train. It combines the ready work, proves the combination once, and wakes an agent only
for a conflict no machine can settle or a failure the proof pins on one chunk. On a repository that batches, what
reaches the branch is exactly the tree that was proven, so two changes that each passed alone cannot land a break
together.

## Left for later

An operator's hand on a batch, abandoning one or ejecting a member by choice, waits with the levers on a branch that
`epic:advanced-delivery/flows` left for later.

## What this epic is not

Keeping chunks likely to touch the same ground from running side by side is not this initiative's.
