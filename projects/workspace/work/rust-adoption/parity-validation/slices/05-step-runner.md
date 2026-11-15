---
name: step-runner
depends-on:
  - fixture-builder
  - state-snapshot
  - comparison
---

# Step runner and the record writer

Builds `runner::run_steps` and `scenario::record` of [record-and-replay.md](../spec/record-and-replay.md), Record and
Runner: running a scenario's steps with a given executable and writing a stable record. Delivers
`engineering:pinned-scenario-environment` and the storing half of `engineering:behavioral-fixtures`. The executable is
whatever the caller passes; tests pass stand-in scripts, and nothing here knows the reference tool.

## Change

- `run_steps` invokes one executable per step under the pinned environment with the step's arguments, working directory,
  standard input, and environment additions, and snapshots state after each step.
- Immediately before each spawn, `run_steps` makes the check of the spec's Runner, "confinement", on the working
  directory it is about to use, and returns `RunFailure::Unconfined` without starting anything when it fails. The check
  does not rely on the model having accepted the scenario.
- `runner.rs` declares `SUPPORTED` as empty and `CALL_CAPTURE` as `false`.
- `record` builds the fixture twice under two roots, runs the steps in each, normalizes, and applies step 4 of the
  spec's "record one scenario": the two runs are compared with `compare::differences` under the scenario's rules, the
  first `Difference` is returned as `RecordError::Unstable`, and otherwise `record/` and `meta.toml` are written.
- `record` applies the `redactions` of its options to everything it stores, after the ROOT replacement, as
  [scenario-format.md](../spec/scenario-format.md), Normalization, says.
- `record` returns `RecordError::Malformed` for a malformed scenario, an unconfined one included, before any tree
  exists, and for `RunFailure::Unconfined`; in both cases it writes nothing.
- An existing `record/` is left untouched unless re-recording is asked for; `expected/` is never read or written.
- Files touched: `crates/test-support/src/scenario.rs`, `crates/test-support/src/scenario/{runner,record}.rs`,
  `crates/test-support/tests/scenario_record.rs`.

## Acceptance

- Recording a two-step scenario with a stand-in script that prints JSON and appends to a workspace file writes
  `steps/01` and `steps/02` with `exit-code`, `stdout`, `stderr`, `stdout.json`, `before/`, and `after/`, and a
  `meta.toml` whose environment and description digests equal the computed ones.
- No file under the written `record/` contains either tree's root path.
- The target executable is never started for an unconfined scenario. The stand-in appends a line to a marker file,
  outside every tree, each time it runs, and each case asserts the marker file does not exist afterwards:
  - `record` on a scenario directory whose step `cwd` is `/tmp`, on one whose `cwd` is `../..`, and on one whose
    checkout `path` is `../outside`: `RecordError::Malformed` naming the key, no `record/`, and no temporary tree was
    created;
  - `run_steps` on a `Scenario` value built in code, bypassing the parser, whose step `cwd` is the absolute path of a
    directory outside the tree, and on one whose `cwd` is `../..`: `RunFailure::Unconfined`;
  - `run_steps` and `record` on a scenario whose `files/` holds a symbolic link `out` to a directory outside the tree
    and whose step has `cwd = "out"`: `RunFailure::Unconfined` and `RecordError::Malformed`, and no `record/`;
  - a two-step scenario whose first step is confined and whose second is not, through `run_steps` on a value built in
    code: the marker holds one line, the result is `RunFailure::Unconfined` for step 2.
- A stand-in that prints its process id as a JSON number, in a step whose stdout mode is `json`, is refused with
  `Unstable` carrying a `Json` difference, and no `record/` exists.
- The same stand-in printing its process id as plain text, in a `text`-mode step, records successfully; so does one that
  writes its process id to stderr.
- A stand-in that writes its process id into a workspace file is refused with a `Files` difference; with a
  `[[compare.file]]` rule of mode `ignore` for that path it records.
- A stand-in that prints its working directory records `<ROOT>/ws` and is stable.
- With a redaction of a directory the test made to the token `<ELSEWHERE>`, a stand-in that prints that directory's path
  on stdout and on stderr records the token in both files and the path in neither.
- Recording over an existing record fails with `Exists` and changes no file; with re-recording asked for it replaces the
  record.
- A step with `stdin` delivers the file's bytes; a step without one reads end of input at once.
- A step's `env` addition reaches the stand-in with `<ROOT>` expanded; a variable set on the test process does not.
- A stand-in that sleeps past the step deadline fails the run and leaves no process behind; the test shortens the
  deadline through a parameter of the internal function `run_steps` wraps.
- `cargo dev gate slice` exits 0.

## Out of scope

- The `cargo dev record` command and its refusals: `record-command`.
- Replay over a scenario set: `replay-target`.
