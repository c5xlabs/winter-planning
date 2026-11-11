# Extension management decisions

## Adoption and enablement policy (decision:agent-ready-operation/extension-management/adoption-policy)

**Decider:** product owner and Workspace engineering

**Open.** Choose whether adoption is one global gate or separate context, projection, and executable controls, including
defaults for manifest-less repositories and double declarations; this blocks finalizing `product:extension-activation`
and `engineering:contribution-provenance`.

## Manifest compatibility (decision:agent-ready-operation/extension-management/manifest-policy)

**Decider:** Workspace and extension-contract engineering

**Open.** Choose manifest evolution, unknown-field handling, legacy aliases, and whether an invalid contribution blocks
the extension or only that contribution, preserving actionable errors rather than permissive coercion; this blocks
finalizing `product:extension-inventory` and `engineering:manifest-validation`.

## Activation state and concurrent changes (decision:agent-ready-operation/extension-management/activation-ownership)

**Decider:** Workspace engineering and the product owner

**Open.** Choose authoritative activation ownership and recovery for removal, source/prefix changes, unavailable
declarations, and competing processes, coordinating artifact ownership with harness projection; this blocks finalizing
`product:extension-reconciliation` and `engineering:activation-transitions`.
