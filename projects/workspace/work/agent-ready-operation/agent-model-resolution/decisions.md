# Agent model resolution decisions

## Canonical compatibility (decision:agent-ready-operation/agent-model-resolution/canonical-compatibility)

**Decider:** product owner and harness engineering

**Open.** Choose omitted-field defaults, unknown-field handling, name-collision severity, and compatibility for
noncanonical agent files without confusing authoring lint with runtime acceptance; this blocks finalizing
`product:canonical-agent-input` and `engineering:native-field-accounting`.

## Native permission policy (decision:agent-ready-operation/agent-model-resolution/permission-policy)

**Decider:** product owner and harness/security engineering

**Open.** Choose whether unsupported permission intent warns, requires acknowledgment, or refuses projection, and what
native validation can establish; this blocks finalizing `product:permission-loss`, not the requirement to expose loss or
the prohibition on claiming equivalence from key presence.

## Bundled model defaults (decision:agent-ready-operation/agent-model-resolution/default-catalog)

**Decider:** product owner and harness engineering

**Open.** Choose the release-owned model defaults and their update/compatibility policy while retaining workspace
remapping and concrete override semantics; this blocks finalizing the bundled defaults for `product:model-precedence`,
not its precedence or provenance requirements.

## Invalid resolution behavior (decision:agent-ready-operation/agent-model-resolution/resolution-failures)

**Decider:** product owner and harness engineering

**Open.** Choose load-time versus per-agent failure boundaries, inspection exit categories, and when higher-priority
concrete overrides may rescue incomplete tiers; this blocks finalizing `product:model-precedence` and
`product:resolution-provenance`, while projection owns retention or withdrawal of prior good artifacts.
