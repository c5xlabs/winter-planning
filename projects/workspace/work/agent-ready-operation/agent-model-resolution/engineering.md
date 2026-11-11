# Agent model resolution engineering requirements

For `epic:agent-ready-operation/agent-model-resolution`.

## What already holds

`engineering:domain-io-boundaries` keeps model policy independent of harness filesystem writes. Configuration layering
belongs to `epic:parallel-workspaces/workspace-configuration`; resolution needs neither installed artifacts nor a live
vendor connection.

## One attributed resolution (engineering:attributed-resolution)

Rendering, inspection, override-target diagnostics, and installed-copy comparison consume the same deterministic model
and effort semantics, including contributing values and their source attribution. Model and effort failures remain typed
per-agent/per-harness results rather than fallback strings. Serves `product:model-precedence`,
`product:effort-precedence`, and `product:resolution-provenance`.

## Distinct override meanings (engineering:override-kinds)

Tier selection, concrete per-harness identifiers, and effort-only profiles remain distinct through resolution; no
content-based guessing converts a concrete identifier into a tier. Native effort is omitted when inherited, and a
profile that changes only the model cannot erase canonical effort. Serves `product:model-precedence` and
`product:effort-precedence`.

## Account for native fields (engineering:native-field-accounting)

Canonical parsing and rendering classify supported common fields, vendor-only fields, ignored or rejected input, and
unsupported permission intent explicitly. Other vendors' override blocks cannot leak into the target artifact, and
permission-loss accounting cannot be suppressed solely by native-key presence. Serves `product:canonical-agent-input`
and `product:permission-loss`.
