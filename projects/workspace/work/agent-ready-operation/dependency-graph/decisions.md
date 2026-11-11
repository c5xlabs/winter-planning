# Dependency graph decisions

## Incomplete graph inputs (decision:agent-ready-operation/dependency-graph/incomplete-inputs)

**Decider:** product owner and engineering

**Open.** Choose between refusing an incomplete graph and returning explicitly partial evidence with an unsuccessful or
qualified outcome, rather than the legacy successful adjacency map that skips broken manifests; this blocks finalizing
`product:dependency-evidence-completeness` and consumers' handling of incomplete graphs.
