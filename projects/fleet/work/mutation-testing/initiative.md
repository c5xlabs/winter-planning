---
initiative: mutation-testing
refinement: refined
epics:
  - diff-scoped
  - corpus-sweep
  - angular
---

# Mutation testing

A machine for the reviewer's doubt about whether the tests would have noticed.

## The problem

A fleet that ships while its owner sleeps acts on one signal: the suite went green. A human reviewer carries a private
doubt about that signal — *would these tests actually have noticed?* — and blizzard's reviewers already act on the doubt
by hand: mutate a suspicious line, re-run the tier, name the assertion that fired, the review practice the workspace's
conventions describe. That practice is judgment applied one line at a time, and it reaches only the lines a reviewer
thought to suspect.

## The outcome

This initiative gives the doubt a machine. Mutation testing seeds small defects — a flipped comparison, a dropped guard,
a negated condition — one at a time, and re-runs the suite against each; a seeded defect the suite survives is a place
the product can regress with the gate still green. Coverage says a line ran; a killed mutant says its behavior is
pinned.

`epic:mutation-testing/diff-scoped` is advisory and in the delivery path, so the feedback reaches the change that
introduced the weak test while it is still open. `epic:mutation-testing/corpus-sweep` folds it into gardening.
`epic:mutation-testing/angular` brings the frontend under the tooling once the tooling can reach it; until then, a
reviewer's hand-picked mutations stay the standing complement to the tooling and the only path open on the frontend
(`decision:mutation-testing/shape`).
