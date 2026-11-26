# Scope environment engineering requirements

For `epic:running-environments/scope-environment`.

## What already holds

`epic:parallel-workspaces/environment-lifecycle` owns authoritative environment identity and index assignment.
`engineering:process-execution`, `engineering:typed-outcomes`, and `engineering:observability-isolation` apply here.
This epic has no dependency on executable-extension delivery: a user-configured value command is not itself a registered
extension. Extension invocation consumes this capability later.

## One computation authority (engineering:scope-computation-authority)

Inspection and workload consumers use one semantic definition of scope, precedence, and interpolation; each consumer
states its lookup-execution policy explicitly. Resolution consumes registry identity without allocating or repairing it;
no consumer derives an index from an unregistered name. Serves `product:runtime-scope-values`,
`product:layered-runtime-values`, and `product:offline-env-inspection`.

## Bounded value execution (engineering:bounded-value-commands)

Value-producing commands have a fixed 30-second deadline per invocation, stdin closed, the workspace root as working
directory, and a visible environment. Each runs in its own process group, and the whole group is killed on timeout or
cancellation. Native argument execution is the default; shell execution requires an operator-authored opt-in and
prohibited substitutions fail before execution. Serves `product:external-value-resolution` and
`product:private-runtime-values`.

## Atomic computation outcomes (engineering:complete-resolution-outcomes)

Computation returns a complete scope result or a typed failure, preserving independently known identity separately from
optional or unresolved values. A template error cannot degrade a launch into an empty environment; diagnostic collection
does not erase the failure cause. Serves `product:complete-launch-environment`.

## Value provenance and confidentiality (engineering:value-provenance)

Resolution retains enough declaration and import provenance to diagnose precedence, blocked references, and prohibited
shell substitutions without recording values in diagnostics or telemetry. Export quoting preserves literal resolved
content; scalar conversion and collision reads follow
`decision:running-environments/scope-environment/resolution-compatibility`. Serves `product:layered-runtime-values` and
`product:private-runtime-values`.
