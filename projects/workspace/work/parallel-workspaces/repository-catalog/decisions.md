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

## Comments removed with a repository (decision:parallel-workspaces/repository-catalog/removal-comment-extent)

**Decider:** product owner

`repo remove` removes the block and its own comments only: the comment lines directly above the removed block go with
it, and the comment lines directly above the next block are kept. The reference tool does the reverse on both counts,
leaving the removed block's comment behind and deleting the next block's, so both halves are a registered deviation.
