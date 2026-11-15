---
name: scenario-model
depends-on:
  - epic:parallel-workspaces/component-runtime
---

# Scenario description types and discovery

Lands the data model of [scenario-format.md](../spec/scenario-format.md) — `scenario.toml`, `reference.toml`,
`record/meta.toml`, facilities, and directory discovery — with nothing running a scenario yet. Delivers the format half
of `engineering:scenario-files` and the declaration half of `engineering:deferred-scenario-accounting`. Of the epic it
depends on, it needs what this epic's engineering requirements list under What already holds.

## Change

- `scenario::model` parses a scenario directory into typed values, rejecting unknown keys at every level, an unknown
  facility, an invalid scenario or command-directory name, a step list that is empty, and `expected/` without
  `[deviation]` or the reverse. `[[stub]]` entries are parsed and validated like every other table.
- The model applies the model checks of the spec's Path confinement — the relative-path rule for every key of its table,
  the one-segment names, and the argument and environment rules — while parsing, and a description that breaks one is
  malformed. The model starts no process and creates no directory.
- Discovery walks a scenarios root by the spec's discovery rule and returns scenario ids sorted, each with its state:
  recorded, unrecorded, or malformed with the reason.
- The description digest is computed as the spec's `record/meta.toml` section defines it.
- `scenarios/reference.toml` is committed with the commit id the spec gives.
- Files touched: `crates/test-support/Cargo.toml`, `crates/test-support/src/{lib,scenario}.rs`,
  `crates/test-support/src/scenario/model.rs`, `crates/test-support/tests/scenario_model.rs`,
  `scenarios/reference.toml`, `Cargo.lock`.

## Acceptance

- A test parses a description using every key of the spec's example and reads each value back.
- For each malformed case above, a test asserts the error names the scenario id and the key or name at fault.
- Path confinement, one test per case, each asserting a malformed result that names the scenario id and the key:
  - a step `cwd` that is absolute (`/tmp/x`);
  - a step `cwd` that leaves the workspace directory: `..`, `../ws`, and `a/../../b`;
  - a fixture path that leaves it: a `[[fixture.checkout]]` `path` of `../outside`, a `[[fixture.worktree]]` `path` of
    `/tmp/x`, a `[[fixture.git]]` `cwd` of `../..`, and a commit `files` key of `../x`;
  - a `[[fixture.remote]]` `name` of `../x`; a `stdin` of `../other/stdin`; a `stdin` that is a symbolic link to a file
    outside the scenario directory;
  - `[[fixture.git]]` `args` starting with `-C`, and `args` holding `--git-dir=/tmp/x`; a step argument starting with
    `/`; a step `env` value starting with `/`.
- A step `cwd` of `a/../b` and a step `env` value of `<ROOT>/ws/x` parse.
- Discovery over a temporary root with three scenario directories under two command directories returns their ids in
  byte order and ignores `reference.toml`.
- The digest is equal for two copies of a scenario, changes when one byte of a file under `files/` changes, and changes
  when a file is renamed without changing its content.
- A test reads the checkout's `scenarios/reference.toml` and gets the 40-character id of the spec.
- `cargo dev gate full` exits 0: the stand-in scenario target still passes, because `scenarios/` holds no directory.

## Out of scope

- Building the fixture, and the resolved half of Path confinement: `fixture-builder`, `step-runner`.
- Any comparison or replay: `comparison`, `replay-target`.
