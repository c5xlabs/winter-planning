---
name: reference-scenarios
depends-on:
  - ws-restack-command
---

# Recorded scenarios of `ws restack`

Records `ws restack` against the reference tool and replays it against the binary, proving the flows of
[restack-plan.md](../spec/restack-plan.md) and [restack-execution.md](../spec/restack-execution.md) end to end under
`decision:functional-parity`; the registered deviation of this epic carries its `[deviation]` decision and expected
result.

## Change

- Scenarios under `scenarios/ws-restack/`, recorded with `cargo dev record` from the reference tool, each with a fixture
  workspace of two projects whose chain branches, amendments, and reflog history are built by `[[fixture.git]]` calls;
  steps store `--json` stdout.
- No fixture declares `nested`, so no other epic's step runs.
- No production code changes.

## Acceptance

- Scenarios: a dry run of a two-link chain; a restack after the bottom predecessor is amended; a squash-landed lower
  branch with `--cut`; a chain already up to date; a dirty worktree refused; an unknown boundary refused; a repeated env
  refused; a conflict in the second repository; a re-run over a chain whose bottom link already completed.
- A deviation scenario, with its decision and `expected/`: BASE `{main}` with a non-participating project that has no
  `main_branch`, refused before any git call, under `decision:parallel-workspaces/stacked-history/token-expansion`.
- `cargo dev gate full` exits 0.
