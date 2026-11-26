---
name: reference-scenarios
depends-on:
  - ws-init-all
  - ws-list
  - ws-destroy-command
---

# Recorded scenarios of `ws init <env>`, `ws list`, and `ws destroy`

Records `ws init <env>`, `ws init --all`, `ws list`, and `ws destroy` against the reference tool and replays them
against the binary, proving the flows of [init.md](../spec/init.md), [list.md](../spec/list.md), and
[destroy.md](../spec/destroy.md) end to end under `decision:functional-parity`; each registered deviation of this epic
carries its `[deviation]` decision and expected result.

## Change

- Scenarios under `scenarios/ws-init/`, `scenarios/ws-list/`, and `scenarios/ws-destroy/`, recorded with
  `cargo dev record` from the reference tool, each with a fixture workspace whose projects are cloned from fixture
  remotes; event order across repositories is compared per target.
- No fixture declares `nested`, an extension with lifecycle hooks, or a provision handler, so the reference runs no step
  another epic owns; a recording that still shows one is cut down until it does not.
- No production code changes.

## Acceptance

- `ws-init` scenarios: an environment created, then reconciled again; a pinned project; a worktree inheriting a
  sibling's upstream; `--all` over two environments.
- `ws-list` scenarios: two environments with feature branches; no environment.
- `ws-destroy` scenarios: a clean environment with `--force`; a dirty one refused; `--dry-run`; two literals with
  `--force`.
- Deviation scenarios, each with its decision and `expected/`: `ws destroy .` and `ws init projects` under
  `decision:parallel-workspaces/environment-lifecycle/env-name-grammar`; a parked worktree under
  `decision:parallel-workspaces/environment-lifecycle/head-state`; a plain directory at a worktree location under
  `decision:parallel-workspaces/environment-lifecycle/location-confinement`; a stray file kept and a missing directory
  released under `decision:parallel-workspaces/environment-lifecycle/teardown-policy`; a registration whose directory is
  gone under `decision:parallel-workspaces/environment-lifecycle/discovery-authority`; a full band and an alias moved
  under `decision:parallel-workspaces/environment-lifecycle/allocation-policy`; an unknown literal under
  `decision:parallel-workspaces/target-selection`.
- `cargo dev gate full` exits 0.
