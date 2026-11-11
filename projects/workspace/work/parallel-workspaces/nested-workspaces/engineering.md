# Nested workspace engineering requirements

For `epic:parallel-workspaces/nested-workspaces`.

## Verified child boundary before every authority change (engineering:child-authority-confinement)

Child-root verification and ancestor-recursion protection remain valid across concurrent path/config changes and every
mutating child invocation; discard approval cannot bypass workspace identity. Process status and output compatibility
are validated without assuming all nonzero exits mean unusable state. Serves `product:verified-child-workspace`.

## Child process environment isolation (engineering:child-process-isolation)

Child invocation excludes enclosing runtime scope, code-selection overrides, and toolchain environment contamination;
only deliberate inherited settings and supported diagnostics/tracing cross the boundary. Arguments are not reconstructed
through a shell. Serves `product:verified-child-workspace` and `product:nested-local-inheritance`.

## Delegation writes are all-validated (engineering:delegation-write-integrity)

Allocation capacity and merged child configuration are validated before any delegation write; concurrent writers and
interruption cannot lose child-owned settings or publish a successful partial delegation. Serves
`product:nested-runtime-allocation` and `product:nested-local-inheritance`.

## Loss audit completeness is load-bearing (engineering:nested-loss-evidence)

Recursive deletion uses complete evidence for all child stores being removed; a skipped standalone probe, missing
tracking ref, failed history read, or incompatible child result cannot count as no local work. Disposable multi-level
Git fixtures cover retained branches, stashes, detached work, and child-service failures. Serves
`product:nested-work-inspection` and `product:nested-teardown-preservation`.
