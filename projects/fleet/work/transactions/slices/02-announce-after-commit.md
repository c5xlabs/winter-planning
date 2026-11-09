---
name: announce-after-commit
---

# Announce after commit

Moves every event a store write announces to the caller that asked for the write, published after commit. Delivers
`engineering:announce-after-commit`.

## Change

- A unit returns what it changed, and its caller publishes from that once the unit commits.
- The publisher a scoped write carries and the callback a store queues on its commit are removed.
- The event the store announces today is published by its caller.

## Acceptance

- No store write calls the broker.
- Each event is published once, after its write commits, and not at all when the write rolls back.
