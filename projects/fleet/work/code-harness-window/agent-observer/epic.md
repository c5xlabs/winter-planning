---
epic: code-harness-window/agent-observer
refinement: outline
status: planned
---

# Agent observer

Part of `initiative:code-harness-window`. Its requirements are inline below.

## The problem

Behind one card on the board, a session running a swarm may carry dozens of agents, some quietly growing past the point
where they still think clearly, and the operator finds out only by asking. A watcher built inside one run dies with it.

## The outcome

A lane on the runner reads each running session's harness transcripts on an interval and reports what it finds, so the
operator sees a swarm's vital signs where they already look, on the chunk's card, and hears when an agent crosses the
line.

Much of the ground is already laid. The runner already samples a running session's context against a warn line and
reports the first crossing, and it already reads the private conversations of the subagents a session spawns when it
ships transcripts to the hub. The observer widens the first to cover the second.

Its spend reading matters beyond the window. A swarm spends its whole budget inside a single node, where today's spend
controls cannot see it; the observer's running count is what would let a ceiling reach inside the node at all, which is
the open question `epic:chokepoint/epic-dwf` leaves for this one to answer.

## Left for later

Whether the observer only reports or may also act, stopping an agent past the line or asking the session to hand it off,
belongs to `epic:steering`. Reporting comes first; acting reaches into steering's territory and is decided there.

## Requirements

### Swarm vital signs (product:swarm-vital-signs)

For each running chunk the observer reports how many agents are live, how much context each one is carrying, which have
crossed the operator's warn line, and how many tokens the session and its agents have spent so far.

Acceptance:

- The chunk's card shows a small reading of its swarm, something like twelve agents live, the heaviest at two hundred
  thirty thousand, three over the line.

### Warn-line crossing (product:agent-warn-line)

An agent crossing the operator's warn line arrives as an event, the way a session's own context warning does today.

## Open questions

- Where a dynamic workflow's agents leave their transcripts. The runner reads the subagents a session spawns directly;
  whether agents launched inside a background workflow run land in the same place is the first thing to establish, and
  the answer decides how much of the observer is reading and how much is new. Engineering can answer.
- Which harnesses can be observed. Each harness keeps its transcripts its own way, so each one answers for its own
  agents behind the harness seam. A harness that cannot run a swarm has little to report beyond its main session.
  Engineering can answer.
