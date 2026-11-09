---
initiative: chokepoint
refinement: refined
epics:
  - chokepoint
  - epic-dwf
---

# Chokepoint

The fleet clears the road for a project's one large change and gives that change a lane built for its size.

## The problem

A few times a year a project has one large thing to do, and everything else has to stop so that it can be done well: a
rewrite from Python into Rust, the board moved from one framework to another, a generation of dependency upgrades, a
tech-debt sweep across the whole codebase. The change behind `epic:architectural-sweep` was one of these. It was safe
only because nothing else was in flight, and it was possible only because dozens of agents worked it in parallel — and
both of those were arranged by hand, from outside the fleet.

## The outcome

The fleet arranges both itself. `epic:chokepoint/chokepoint` clears the road: the operator marks a chunk as a
chokepoint, sees it flagged on the board well before it arrives, moves it up or down the queue as plans change, and the
fleet finishes everything ranked ahead of it, works it alone, and holds back the work behind it until it clears.
`epic:chokepoint/epic-dwf` gives the work a lane built for its size: a development workflow for epic-level change of
twenty thousand lines or more, or thousands of small adjustments, planned whole, built by a swarm of agents, and landed
as one.
