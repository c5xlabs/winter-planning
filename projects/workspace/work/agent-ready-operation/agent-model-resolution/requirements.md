# Agent model resolution requirements

For `epic:agent-ready-operation/agent-model-resolution`.

## Read a canonical agent (product:canonical-agent-input)

A harness engineer can author one agent definition and distinguish its common intent from harness-native settings.

Acceptance:

- Canonical identity, description, model tier, tools, body, and per-harness overrides are recognized with actionable
  diagnostics for malformed input.
- Flat Markdown agent files and the exclusion of documentation-only entries are explicit; unsupported nested agent
  layouts do not misleadingly appear installed.
- The policy for omitted names/default model fields is visible and distinguishes runtime compatibility from stricter
  authoring guidance under `decision:agent-ready-operation/agent-model-resolution/canonical-compatibility`.
- Duplicate canonical identities are reported across extensions, even when installation prefixes differ.

## Resolve models deliberately (product:model-precedence)

An engineer can remap a tier or target one agent without editing extension source, and can predict which setting wins.

Acceptance:

- Model precedence is agent-specific workspace override, canonical harness model, configured tier mapping, then built-in
  tier default; personal versus shared config merging follows the workspace configuration contract.
- Built-in `fable`, `opus`, `sonnet`, and `haiku` tiers and custom tiers can map separately to Claude Code, Codex, and
  OpenCode model identifiers; overriding a built-in vendor cell retains its other built-in cells.
- A bare agent override string selects a tier for all harnesses; a per-harness string or profile model is a concrete
  identifier, even if its spelling matches a tier.
- Unknown tier labels, missing vendor mappings, malformed overrides, and overrides naming no known agent are diagnosed
  separately, with their affected agents and harnesses.
- Resolution does not claim that a configured identifier exists, is available to the user, or has a vendor-certified
  capability rank.

## Resolve reasoning effort separately (product:effort-precedence)

An engineer can adjust an agent's reasoning effort without accidentally replacing its model or discarding unrelated
harness settings.

Acceptance:

- A per-harness profile can set model, effort, or both; effort-only configuration preserves ordinary model resolution.
- Workspace profile effort wins over canonical native effort; absent profile effort preserves canonical effort, and an
  absent effort at both layers means harness/session inheritance rather than an invented explicit default.
- Effective effort projects to Claude `effort`, Codex `model_reasoning_effort`, and OpenCode `reasoningEffort`.
- Effort strings are shown as configured native values, not represented as validated vendor support.

## Explain effective settings (product:resolution-provenance)

A person or agent can inspect the same resolution used for installation, including why their personal setting won.

Acceptance:

- Inspection exposes effective tier mappings, configured agent overrides including unmatched targets, and each known
  agent's per-harness model and effort.
- Results identify built-in defaults, configured tier values, canonical harness overrides, agent overrides, and the
  shared/local source of configured values; non-applicable and failed resolution are distinguishable.
- Editing configuration is visible on the next inspection without restarting a long-lived caller.
- Inspection remains read-only and can show a cell-level error without hiding other cells or presenting it as a
  configuration-load success for the failed cell.
- Installed-copy state, when requested, comes from `epic:agent-ready-operation/harness-projection`, without a separate
  model resolution policy in the drift checker.

## Expose permission loss (product:permission-loss)

An engineer learns when a canonical tool restriction does not protect an agent on another harness. A native setting is
not described as equivalent merely because its key exists.

Acceptance:

- Common tools retained for Claude but unsupported by Codex/OpenCode are reported as a projection loss.
- Native sandbox/permission settings are distinguishable from the common tool list and from inherited access.
- Presence of `sandbox_mode` or `permission` alone does not establish equivalent restrictions or certify safety.
- Installation or refusal follows an explicit loss policy owned by
  `decision:agent-ready-operation/agent-model-resolution/permission-policy`; unknown native settings are not invented or
  translated into a broader grant without notice.
