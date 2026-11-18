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
`product:configuration-validation` as a whole and closes the epic's coverage of the spec's Load, Layering, and Data
sections, under `decision:functional-parity`.

## Change

- One unit test per pair of consecutive Load steps that can both refuse, each with a fault at both steps, asserting the
  earlier step's error.
- Recorded `ws index` scenarios under `scenarios/ws-index/`, recorded from the reference tool, each a configuration the
  load refuses: a removed key in the local file only; an invalid `adopt_extensions`; two standalones with the same name;
  a `WINTER_` command entry; an override naming an unknown tier; a configuration with faults at steps 1 and 5. Each
  asserts exit 1, an empty stdout, and an unchanged tree.
- A recorded scenario for the registered deviation, `git_excludes = "abc"`, whose expectation is the binary's refusal
  under `decision:parallel-workspaces/workspace-configuration/element-type-validation`, not the reference tool's
  acceptance.
- `context/verification/matrix.md` gains the rows of the new scenarios.

## Acceptance

- The step-order unit tests pass for every such pair.
- Each recorded scenario passes replay, and the deviation scenario is listed as a registered deviation.
- `cargo dev gate full` exits 0.

## Out of scope

- Scenarios of commands that write configuration: their own epics.
