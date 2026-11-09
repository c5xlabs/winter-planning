---
epic: lab/seeded-defects
refinement: outline
status: planned
---

# Seeded defects

Part of `initiative:lab`; its requirements are not yet written.

## The problem

The hardest thing to measure about a gate is what it misses, because in ordinary work no one knows what was there to
catch. Nobody can say today how often the review gate catches a real bug.

## The outcome

The lab manufactures that knowledge: it plants known defects into a specimen's artifacts, a bug in a diff on its way to
`review` or a missing requirement in a plan on its way to `plan-review`, and measures how often each arm catches them.
It is the same move `initiative:mutation-testing` makes against a test suite, made against the fleet's own reviewers,
and it yields a number no one has today: how often the review gate catches a real bug.
