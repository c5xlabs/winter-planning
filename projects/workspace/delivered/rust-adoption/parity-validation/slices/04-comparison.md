---
name: comparison
depends-on:
  - state-snapshot
---

# Comparison of a record with an observation

Builds `scenario::compare`: the Comparison section of [record-and-replay.md](../spec/record-and-replay.md), over
in-memory values. Delivers the replay-failure, message-text, human-text, and deviation bullets of
`engineering:behavioral-fixtures` under `decision:functional-parity` and
`decision:rust-adoption/parity-validation/state-comparison`.

## Change

- `compare::differences` takes a scenario's rules, the recorded steps, and the observed steps, both in the in-memory
  form of `record/steps/**` (`StepRecord`, defined here), and returns the `Difference` values in the dimensions `Exit`,
  `Json`, `Tree`, `Files`, and `Git`.
- JSON comparison applies the message-text and fan-out rules; file comparison applies the per-path mode.
- Files under `expected/steps/<nn>/` replace the recorded ones before comparison.
- `StaleRecord` and `Fixture` differences are produced from `meta.toml` and the first `before/` snapshot.
- Text of stdout and stderr produces no difference in any mode.
- Files touched: `crates/test-support/src/scenario.rs`, `crates/test-support/src/scenario/compare.rs`,
  `crates/test-support/tests/scenario_compare.rs`.

## Acceptance

- Equal inputs yield no difference.
- One test per dimension changes one thing and asserts exactly one `Difference` with that dimension and subject: the
  exit code; one JSON value, with the pointer as subject; an extra tree entry; one line of one `git.txt` block, with
  repository and block as subject.
- One changed byte of a `bytes` file yields one `Files` difference and one `Tree` difference for that path, and no
  other.
- A string at a `message_text` pointer that differs yields no difference; a number there instead of a string yields one.
- Two fan-out runs with partitions interleaved differently are equal; a reordering inside one partition is a difference;
  a value outside the run that moves is a difference.
- A file in `toml` mode with reordered keys is equal and with a changed value is not; `presence` accepts any content and
  fails on absence; `ignore` accepts absence.
- Different stderr and different `text`-mode stdout yield no difference.
- With an `expected/` exit code that differs from the record, an observation equal to the expectation yields no
  difference and one equal to the record yields `Exit`.
- A `meta.toml` naming another reference commit, another environment digest, or another description digest each yields
  `StaleRecord`.
- `cargo dev gate slice` exits 0.

## Out of scope

- The `Calls` dimension: `epic:agent-ready-operation/executable-extensions`.
- Running anything: `step-runner`.
