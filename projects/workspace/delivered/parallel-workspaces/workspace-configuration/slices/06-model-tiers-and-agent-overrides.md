---
name: model-tiers-and-agent-overrides
depends-on:
  - load-seam
---

# Model tiers and agent overrides are read and refused at load

Fills `config_model/agents.rs` with the `model_tiers` and `agent_model_overrides` rows and the built-in tier table of
[configuration-model.md](../spec/configuration-model.md). Serves `product:configuration-validation`.

## Change

- The effective tier table is the built-in table overlaid per vendor cell by `[model_tiers]`; each `ModelTierFault` is
  refused. A model id of white space alone is accepted here.
- A `model_tiers` that is not a table reads as absent, the built-in table alone, with no message, as the spec's block
  under Tables states from the reference source.
- Each agent override is a tier label of the effective table or a vendor table; each `AgentOverrideFault` is refused,
  `UnknownTier` carrying the known labels.
- Recorded scenarios, each plain, over fixture workspaces in temporary directories:
  `ws-index/model-tier-unknown-vendor`, `ws-index/model-tier-no-vendor`, `ws-index/agent-override-unknown-tier`,
  `ws-index/agent-override-added-tier`, `ws-index/agent-override-unknown-profile-key`, and
  `ws-index/model-tiers-is-string`, whose shared file alone holds `model_tiers = "x"`.
- Files touched: `crates/core/src/config_model/agents.rs`; `scenarios/ws-index/**` for the six scenarios.

## Acceptance

- A unit test asserts the twelve cells of the built-in table against the literal values of the model.
- Unit tests assert each `ModelTierFault` and each `AgentOverrideFault` with its carried values, that a local
  `[model_tiers]` label replaces the shared label whole, and that a white-space model id passes in a tier and is refused
  in an override.
- A unit test asserts that `model_tiers = "x"` gives the built-in table and no error.
- `cargo scenarios ws-index` passes, the six scenarios against the record of the reference tool with no deviation. The
  record of `model-tiers-is-string` shows exit code 0; if it shows exit code 1, the change stops and reports it.
- `cargo dev gate slice` passes; no file outside those named changes.

## Out of scope

- Resolving a model for an agent, which belongs to `epic:agent-ready-operation/agent-model-resolution`.
