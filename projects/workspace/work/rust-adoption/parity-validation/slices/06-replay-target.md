---
name: replay-target
depends-on:
  - step-runner
---

# Replay of every scenario, as the full gate's last step

Builds `runner::replay_all` and replaces the stand-in scenario test target with it:
[record-and-replay.md](../spec/record-and-replay.md), Replay and Runner. Delivers the replay bullets of
`engineering:behavioral-fixtures`, `engineering:deferred-scenario-accounting`, the discovery bullet of
`engineering:scenario-files`, and the scenario clause of
`engineering:parallel-workspaces/component-runtime/verification-gates`.

## Change

- `replay_all` discovers scenarios, defers those whose `needs` the runner cannot satisfy, runs the rest on worker
  threads each in its own tree, compares, and prints the per-scenario lines and the summary in id order. It passes no
  extra environment to `run_steps`.
- An unrecorded, malformed, or stale scenario, a scenario with a `[[stub]]`, a fixture that cannot be built, and a
  timeout each fail the run.
- `crates/cli/tests/scenarios.rs` calls `replay_all` with the binary Cargo built and the repository's `scenarios/` root,
  accepts filters and `--list`, and exits 0 or 1.
- `WINTER_TEST_KEEP=1` keeps the tree of each failed scenario; `WINTER_SCENARIO_JOBS` sets the worker count.
- Files touched: `crates/test-support/src/scenario/runner.rs`, `crates/test-support/tests/scenario_replay.rs`,
  `crates/cli/tests/scenarios.rs`.

## Acceptance

- Over a temporary scenarios root holding three scenarios recorded with a stand-in script, replay with the same script
  reports three `ok` lines in id order and `scenarios: 3 passed, 0 failed, 0 deferred`.
- Replay with a script that exits with another code for one scenario reports `FAILED` for it with an `Exit` block, `ok`
  for the others, and a failed report.
- A scenario with `needs = ["pty"]` is reported `deferred <id> needs pty`, is not run (the stand-in records no
  invocation), and does not fail the run.
- A scenario directory without `record/` fails the run and is reported unrecorded; a malformed one fails naming the key;
  one with a `[[stub]]` fails with a `Malformed` difference whose subject is `stub`.
- A filter runs only matching ids; `--list` prints every id with its state and runs nothing.
- With eight scenarios and one worker, and with eight workers, the printed output is identical.
- `cargo scenarios` on the checkout exits 0 with `scenarios: 0 passed, 0 failed, 0 deferred`; `cargo dev gate full`
  exits 0.

## Out of scope

- Recording against the reference tool: `record-command`.
- Stub programs and the `Calls` dimension: `program-capture`.
