---
epic: lab/bench
refinement: outline
status: planned
---

# Bench

Part of `initiative:lab`; its requirements are in `requirements.md`.

## The problem

A question about one stretch of a graph, such as whether a new plan-review prompt catches weak plans, cannot be answered
by running whole chunks: everything upstream varies along with the thing under study. `persona:harness-engineer` needs
to run only the node-steps the question is about, with everything before them held fixed and everything after them cut
away.

## The outcome

An experiment is a graph, derived from the one under study. The engineer points at an existing graph, say `adv-dwf`, and
names the **bench**: the node-steps under study and the loop between them. For a question about planning, the bench is
`plan`, `plan-review`, and the revise edge that sends a plan back for another pass. The lab transforms the source graph
into a lab graph around it:

```text
adv-dwf:  plan ⇄ plan-review → build → verify → … → deliver → … → done

lab:      [seed] → plan ⇄ plan-review → ◼ end state (would-have-built)
```

Three rewrites carry the whole transformation. The bench is kept verbatim, so what runs is what the fleet really runs.
Everything upstream of the bench is replaced by a seed. And every edge leaving the bench is cut and capped with an **end
state**. `build` and `deliver` are never run; they are hijacked into end states named for what would have happened next.
The bench can be any connected stretch: `review` alone, fed one fixed diff, or a wider bench such as `build ⇄ verify`,
which answers a different question. The lab neither knows nor cares which question is being asked; it knows only which
nodes to keep, what to seed, and where to cut.

The transformation is useful before anything else in the initiative exists. With a seed written by hand, an engineer can
start a trial chunk on each arm by hand and read the end states side by side. Paired with `epic:demo`'s mock harness, a
lab graph can be run for free, which makes the transformation itself testable without spending a token.
