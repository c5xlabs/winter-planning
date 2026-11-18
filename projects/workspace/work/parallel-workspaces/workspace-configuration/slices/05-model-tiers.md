---
name: model-tiers
depends-on:
  - layered-load
code:
  - winter-workspace/crates/core/src/config.rs
  - winter-workspace/crates/core/src/termination.rs::ConfigError
---

# The model tiers and agent overrides

Adds to the load the `model_tiers` and `agent_model_overrides` rows of Tables, the built-in model tiers table, and the
`UnknownTier` cross-key rule of [configuration-model.md](../spec/configuration-model.md), at the end of step 8 of the
Load order. Delivers the operation-relevant and closed-set bullets of `product:configuration-validation` for these two
tables. Resolving an agent's model stays with `epic:agent-ready-operation/agent-model-resolution`.

## Change

- `ConfigError` gains `ModelTier` and `AgentOverride`, with `ModelTierFault` and `AgentOverrideFault` as the spec's
  Domain gives them.
- The effective tier table is the built-in table overlaid per vendor cell by `[model_tiers]`. A `[model_tiers]` label
  that is not a vendor table, an unknown vendor, a model id that is not a non-empty string, and a label with no vendor
  are each their `ModelTierFault`.
- An `agent_model_overrides.<agent>` string must be a label of the effective tier table, else `UnknownTier` carrying the
  known labels; an empty value, a wrong type, an unknown vendor, an empty vendor table, a vendor value of the wrong
  type, and each profile fault are their `AgentOverrideFault`. Profile keys are closed to `model` and `effort`, with at
  least one set.
- The built-in tiers exist only as the labels and cells the check needs. No value is kept.

## Acceptance

- Unit tests, one per `ModelTierFault` variant: `[model_tiers] fast = "x"`; `fast = { gemini = "g" }`;
  `fast = { claude = 1 }`; `fast = {}`.
- Unit tests, one per `AgentOverrideFault` variant, each naming the agent: `a = "turbo"` lists the four built-in labels;
  `a = ""`; `a = 1`; `a = { gemini = "g" }`; `a = {}`; `a = { claude = "" }`; `a = { claude = 1 }`;
  `a = { claude = { size = 1 } }`; `a = { claude = { model = 1 } }`; `a = { claude = { effort = 1 } }`;
  `a = { claude = {} }`.
- Unit tests: `a = "opus"` loads; with `[model_tiers] fast = { claude = "c" }`, `a = "fast"` loads; a label defined only
  in the local file is known to an override in the shared file.
- `cargo dev gate full` exits 0.

## Out of scope

- Choosing a model for an agent and the `agents` command: `epic:agent-ready-operation/agent-model-resolution`.
