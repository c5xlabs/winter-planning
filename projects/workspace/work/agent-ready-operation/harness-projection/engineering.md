# Harness projection engineering requirements

For `epic:agent-ready-operation/harness-projection`.

## What already holds

`engineering:concurrent-state-safety` governs accepted updates and recoverable writes. Context source/address decisions
belong to `epic:agent-ready-operation/context-navigation`; the resolver in
`epic:agent-ready-operation/agent-model-resolution` is independent of installed artifacts, avoiding a dependency cycle.

## Destination authority before mutation (engineering:projection-authority)

Projection validates the complete desired namespace and source/destination containment against explicit ownership before
mutation, including equal/dash-overlapping prefixes, name aliases, and workspace/extension collisions. Symlinks are
inspected without writing through them; legacy agent-link replacement cannot modify its canonical target. Serves
`product:projection-ownership`.

## Reproducible transformation (engineering:projection-equivalence)

Installation and inspection share naming, source selection, transforms, and expected-content semantics, including
OpenCode's installed skill name and inherited effort omission. Relative symlink targets remain relocatable with the
workspace where applicable, and unchanged output produces no timestamp churn. Serves `product:skill-discovery`,
`product:agent-discovery`, and `product:projection-drift`.

## Render preflight and write accounting (engineering:projection-failure-accounting)

All requested vendor renders for an agent complete before its first write; render preflight is not represented as a
filesystem transaction. Copy replacement, ownership updates, and interruption recovery preserve prior valid content or
leave a diagnosed recoverable state, with no destructive fallback from unavailable sources or malformed input. Serves
`product:projection-recovery`.

Acceptance:

- Failure injection covers the first and later vendor writes, copied-skill replacement, and ownership-update failure.
- Repeated repair does not write through source links, erase unknown destinations, or lose another accepted update.
- Competing processes are exercised against the chosen ownership policy, not only concurrent tasks in one process.

## Context file boundaries (engineering:context-write-authority)

Context projection edits only its authorized generated file or region and preserves authored entry points and local
instructions. Empty extension sets, disabled adoption, malformed declarations, and malformed legacy markers are explicit
transitions rather than shortcuts that skip cleanup or broaden eager imports. Serves `product:harness-context-entry`.
