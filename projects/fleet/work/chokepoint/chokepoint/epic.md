---
epic: chokepoint/chokepoint
refinement: refined
status: planned
---

# Chokepoint

Part of `initiative:chokepoint`; the product requirements are in `requirements.md`.

## The problem

Every so often a project has one large thing to do, and everything else has to stop before it can be done well. A
rewrite of one component from Python into Rust, the board moved from one framework to another, a round of dependency
upgrades, a tech-debt sweep that touches every corner of the codebase — the work behind `epic:architectural-sweep` ran
this way, dozens of agents in parallel across one enormous change, and it was only safe because nothing else was in
flight. Work that moves that much ground cannot share the road. A chunk running beside it lands on code that shifted
underneath it, and the merge that reconciles the two is exactly where functionality quietly goes missing.

Today the operator who knows this has to enforce it by hand: watch the queue, hold back new work, wait for the fleet to
drain, start the big change, and remember to let everything else go once it lands.

## The outcome

A chokepoint lets the operator say it once, on the chunk itself, and lets the fleet keep the discipline for them. The
name is borrowed from the narrow pass every army must cross single file: the fleet narrows toward the chunk, passes
through it alone, and spreads back out once it clears. Whether the big change happens now or later is the operator's to
decide by where they place it; the fleet only keeps the rules each time it reaches for work.

## What this epic is not

A chokepoint does not police dependencies. A chunk that names the chokepoint as a prerequisite was behind it anyway, and
a chokepoint that names a chunk ranked behind it as a prerequisite will wait for it forever. Blizzard does not guard
against that; the board shows what the chokepoint is waiting on, and seeing the mistake is the operator's job.
