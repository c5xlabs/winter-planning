# Mutation testing decisions

## Equivalent mutants (decision:mutation-testing/equivalent-mutants)

**Open.** The equivalent-mutant policy: a suppression list, a score threshold, or per-file ignores. It must be decided
before any mutation run is allowed to block.

## The shape of mutation testing (decision:mutation-testing/shape)

Four shapes were weighed before the initiative was cut into its epics:

1. **Diff-scoped, advisory, in the delivery path** — chosen as the first epic, `epic:mutation-testing/diff-scoped`: the
   feedback reaches the change that introduced the weak test, while it is still open.
2. **A full-corpus sweep only** — kept, but second, as `epic:mutation-testing/corpus-sweep`, and folded into gardening
   rather than built as its own schedule: its findings are disconnected from any open change, and a first run surfaces
   hundreds of survivors that need a standing triage policy before they are signal.
3. **Agent-driven selection only, no tool** — formalizing `bzh:mutation-review-selection` into a review-time step and
   stopping there. Highest signal per mutant and zero new dependencies, but it only reaches lines someone suspected; it
   stays as the standing complement to the tooling, and as the only path currently open on the frontend
   (`epic:mutation-testing/angular`).
4. **A full-corpus blocking gate** — rejected: the runtime would sit in every delivery's path for a signal that mostly
   concerns unchanged code.
