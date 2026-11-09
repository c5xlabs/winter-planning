---
epic: code-harness-window/swarm-view
refinement: outline
status: planned
---

# Swarm view

Part of `initiative:code-harness-window`. Its requirements are not yet written.

## The problem

Knowing that a swarm is healthy does not tell the operator what it is doing. Behind one card, a session may have
launched several workflow runs, each in its own phase, each with agents of its own, and the board shows none of that
structure.

## The outcome

The swarm view is the larger window `epic:code-harness-window/agent-observer` opens onto: the swarm's structure rather
than its vital signs. It shows which workflow runs a session has launched, the phase each is in, which agents belong to
which run, and the session's own running record of where the work stands.

It is deliberately second. The observer answers whether the swarm is healthy; the swarm view answers what it is doing,
and it is worth building only once the observer has shown which questions operators actually ask.
