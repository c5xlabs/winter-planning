# Dependency graph engineering requirements

For `epic:agent-ready-operation/dependency-graph`.

## Declaration-only reads (engineering:graph-read-authority)

Graph discovery reads the effective repository and extension declarations without evaluating contribution programs or
performing reconciliation. Failed discovery retains a distinct cause from a module that legitimately declares no edges.
Serves `product:declared-module-dependencies` and `product:dependency-evidence-completeness`.

## Source identity and deterministic evidence (engineering:graph-source-identity)

Graph results retain enough source identity to disambiguate duplicate module names, unavailable targets, and unreadable
declarations; a name collision cannot silently overwrite a node. Unchanged inputs yield equivalent ordered results
independently of filesystem enumeration order. Serves `product:dependency-query-consumption`.

## Completeness survives consumption (engineering:graph-completeness-propagation)

Every supported graph-consumption path preserves the completeness outcome selected by
`decision:agent-ready-operation/dependency-graph/incomplete-inputs`; an incomplete query cannot become successful
complete evidence inside a check. Dependency rule enforcement remains separate from collecting declared edges. Serves
`product:dependency-evidence-completeness`.
