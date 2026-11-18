---
name: load-order-proof
depends-on:
  - top-level-refusals
  - repository-entries
  - env-bands
  - model-tiers
code:
  - winter-workspace/crates/core/src/config.rs
---

# The whole load, proved against the reference tool

Proves the Load flow of [configuration-model.md](../spec/configuration-model.md) end to end: the first refusal met ends
the load, in the stated order, and the binary's refusals match the reference tool's outcome. Delivers
`product:configuration-validation` as a whole, under `decision:functional-parity`.

## Change

- One unit test per pair of Load steps that can each refuse with no refusing step between them, each with a fault at
  both steps, asserting the earlier step's error; the pre-merge boundaries count as steps: shared file before local,
  `AgentOverridesNotTable` before the merge refusal, and the merge refusal before step 1.
- Recorded `ws index` scenarios under `scenarios/ws-index/`, recorded from the reference tool, each a configuration the
  load refuses: a removed key in the local file only; an invalid `adopt_extensions`; two standalones with the same name;
  a `WINTER_` command entry; an override naming an unknown tier. Each asserts exit 1 and an unchanged tree.
- A recorded `scenarios/repo-remove/` scenario with `[fixture] workspace_git = true`, recorded from the reference tool:
  `repo remove` of a declared repository with `session_prefix` set exits 1 and leaves the tree unchanged.
- A recorded `ws index` scenario of `git_excludes = "abc"`, registered as "Register a deviation" in
  `context/verification/scenarios.md` says: `scenario.toml` gains
  `[deviation] decision =
  "decision:parallel-workspaces/workspace-configuration/element-type-validation"`, and
  `expected/` holds the binary's refusal in place of the reference tool's acceptance.
- `context/architecture/structure.md`: the `config.rs` row describes the module as it stands after the last refusal.

## Acceptance

- The step-order unit tests pass for every such pair.
- Each recorded scenario passes replay.
- `cargo test -p winter-workspace-dev` exits 0 with the edited `structure.md`.
- `cargo dev gate full` exits 0.

## Out of scope

- Scenarios of commands that write configuration, beyond the `repo remove` refusal: their own epics.
