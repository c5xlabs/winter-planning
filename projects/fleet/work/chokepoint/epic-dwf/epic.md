---
epic: chokepoint/epic-dwf
refinement: refined
status: planned
---

# Epic-level development workflow

Part of `initiative:chokepoint`; the product requirements are in `requirements.md`.

## The problem

Every lane the fleet has today is built around one agent at a time. A chunk travels its graph node by node, and at each
node a single session does the work, however large the work turns out to be. That suits the feature-sized change most
chunks carry. It does not suit the change a project makes a few times a year: rewriting a component from Python into
Rust, moving the board from one framework to another, upgrading a generation of dependencies, or clearing out a body of
tech debt across the whole codebase. The work behind `epic:architectural-sweep` was one of these. It reached its tens of
thousands of lines because dozens of agents worked in parallel against one plan, and it was driven by hand from outside
the fleet, because no graph blizzard ships could have carried it.

## The outcome

That work gets a lane of its own: a graph, `epic-dwf`, built for epic-level development. It sits beside `bas-dwf` and
`adv-dwf` as the heaviest of the development workflows, and the work it is meant for comes in two shapes.

The first is **epic-level development** — a change measured in twenty thousand lines or more, too large for one session
to hold and too interlocked to split into separate chunks that land on their own. Here the graph plans the change as a
whole and sets a swarm of agents to it.

The second is **the swarm of small adjustments** — work that is broad rather than deep. A dependency bump that ripples
through four hundred call sites, a new rule that every module must now obey, a rename carried through the whole
codebase: no single edit is hard, but there are thousands of them, and each is best made by an agent looking only at its
own corner. Here the graph's value is less in the planning than in the fan-out.

Both shapes share the property that made the architectural sweep safe: they can only run when nothing else is moving. An
`epic-dwf` chunk is, in practice, always a chokepoint (`epic:chokepoint/chokepoint`), and the two are meant to be used
together — the chokepoint clears the road, and the swarm uses all of it. The lane is meant to get better at being a
swarm every time it runs.
