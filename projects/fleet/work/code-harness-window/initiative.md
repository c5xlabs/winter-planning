---
initiative: code-harness-window
refinement: outline
epics:
  - agent-observer
  - swarm-view
---

# Code harness window

The code harness window shows the operator what a coding harness is doing while it works: the context its session
carries, the agents it has running, and how each of them is doing.

## The problem

An operator watching a chunk on the board sees one card, one session, one node. For most chunks that is the truth. For a
chunk running a swarm it is barely a summary: behind that one card, a single session may be running dozens of agents at
once, each with a context of its own, some of them quietly growing past the point where they still think clearly. The
architectural sweep ran about four hundred and fifty agents through one orchestrating session, and its first generation
grew to more than half a million tokens of context before anyone noticed. The operator caught it by asking, and the
orchestrator answered by building a watcher of its own that polled every agent each minute and raised a hand at a
threshold. That watcher was the right idea, built in the wrong place: it lived inside one run and died with it.

## The outcome

The operator has a window into the coding harness while it works: what context the session is carrying, which agents it
has running, and how each of them is doing. `epic:code-harness-window/agent-observer` comes first: a lane on the runner
that reads each running session's agents and reports their vital signs to the board.
`epic:code-harness-window/swarm-view` follows it with the swarm's structure: which runs a session has launched, the
phase each is in, and which agents belong to which. The observer answers whether the swarm is healthy; the swarm view
answers what it is doing.

## Stance

The window keeps the shape blizzard already holds to. The hub cannot reach into a runner, and should not; the runner is
the one that looks, and what it sees travels up as facts like everything else it reports, so the board renders a swarm
the same way it renders a lease.
