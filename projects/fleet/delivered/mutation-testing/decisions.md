# Mutation testing decisions

## Equivalent mutants (decision:mutation-testing/equivalent-mutants)

**Open.** The equivalent-mutant policy: a suppression list, a score threshold, or per-file ignores. It must be decided
before any mutation run is allowed to block.

## The shape of mutation testing (decision:mutation-testing/shape)

Mutation testing starts diff-scoped and advisory in the delivery path (`epic:mutation-testing/diff-scoped`), because its
feedback reaches the change that introduced the weak test while that change is still open. A full-corpus sweep follows
second, folded into gardening (`epic:mutation-testing/corpus-sweep`), agent-driven selection stays the standing
complement, and a full-corpus blocking gate is rejected; the shapes weighed are in
[mutation-testing shapes](../../research/mutation-testing/shapes.md).
