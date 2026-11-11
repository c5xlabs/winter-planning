# Repository catalog decisions

## Repository identities (decision:parallel-workspaces/repository-catalog/repository-identities)

**Decider:** product owner and engineering

**Open.** Choose intentional dual-role declarations, source aliases, implicit repository discovery, and extension
eligibility precedence; this blocks finalizing `product:repository-roles` and duplicate handling in
`product:edit-repository-inventory`, not the distinction between declaration and materialization.

## Managed locations (decision:parallel-workspaces/repository-catalog/location-policy)

**Decider:** product owner and engineering

**Open.** Choose supported relative layouts, reserved names, intentional overlaps, and whether contained symlinks are
supported or refused; this blocks finalizing `product:repository-addresses` and `engineering:catalog-path-confinement`,
while traversal and escapes remain unsafe in every option.
