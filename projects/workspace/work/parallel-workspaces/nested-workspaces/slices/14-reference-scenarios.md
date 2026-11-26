---
name: reference-scenarios
depends-on:
  - nested-init-step
  - status-nested-member
  - child-retirement
---

# Recorded scenarios of the nested refusals

Records the nested paths of `ws init <env>`, `ws status`, and `ws destroy` that start no child CLI against the reference
tool and replays them against the binary, under `decision:functional-parity`; each registered deviation among them
carries its `[deviation]` decision and expected result. A path that starts a child CLI is proved by
`multi-level-fixtures` instead: the scenario runner runs no `winter` but the one under test or record, so a child call
in a recorded tree would not be the reference's own.

## Change

- Scenarios under `scenarios/ws-init/`, `scenarios/ws-status/`, and `scenarios/ws-destroy/`, recorded with
  `cargo dev record`, each with a fixture workspace declaring one nested project whose env worktree holds no
  `.winter/config.toml`, or a step `env` setting `WINTER_NESTED_CHAIN` through `<ROOT>`.
- No production code changes.

## Acceptance

- Scenarios: `ws init alpha` over a nested project without a marker fails that repository and creates the others;
  `ws status --json` reports the member's error and exits 1; `ws destroy alpha` refuses on the unreadable child.
- A scenario: `ws init alpha` with `WINTER_NESTED_CHAIN` holding the workspace root refuses the nested repository
  without spawning.
- A deviation scenario, with its decision and `expected/`: `ws destroy alpha --force` over the unreadable child keeps
  the env under `decision:parallel-workspaces/nested-workspaces/recursive-teardown`, where the reference removes it.
- `cargo dev gate full` exits 0.
