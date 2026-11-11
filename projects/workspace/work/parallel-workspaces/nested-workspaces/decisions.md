# Nested workspace decisions

## Child component compatibility (decision:parallel-workspaces/nested-workspaces/child-compatibility)

**Decider:** product owner, Platform engineering, and Workspace engineering

**Open.** Choose parent/child version skew, child executable selection, inspection compatibility, and recursion limits;
this blocks finalizing `product:verified-child-workspace` and `product:nested-work-inspection` without freezing legacy
status schema as a child protocol or allowing Python delegation.

## Delegation identity and ownership (decision:parallel-workspaces/nested-workspaces/delegation-policy)

**Decider:** product owner and engineering

**Open.** Choose stable versus config-order child allocations, parent capacity enforcement, inheritance defaults, and
ownership/retraction of generated child overrides; this blocks finalizing `product:nested-runtime-allocation` and
`product:nested-local-inheritance`, not footprint validation or preservation of unrelated child settings.

## Recursive teardown policy (decision:parallel-workspaces/nested-workspaces/recursive-teardown)

**Decider:** product owner and engineering

**Open.** Choose all-child preflight, failure continuation, deeply nested unknown state, and consent for removing entire
child Git stores; this blocks finalizing `product:nested-teardown-preservation` and `engineering:nested-loss-evidence`,
with resource/service semantics owned by their runtime epics.
