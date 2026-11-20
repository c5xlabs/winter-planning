---
name: proof-state-changing
depends-on:
  - remove-command
---

# Proof on a state-changing behaviour: repo remove recorded and replayed

Records the four `repo-remove` scenarios of [method-proof.md](../spec/method-proof.md), Proof scenarios, against the
reference tool, writes the expectation of the one registered deviation, replays all four with the binary, and proves a
wrong implementation is caught. Delivers the state-changing half of `engineering:method-proof`, with the edited file
compared by bytes under `decision:rust-adoption/parity-validation/state-comparison` and the deviation under
`decision:parallel-workspaces/repository-catalog/removal-comment-extent`. The reference tool runs once per scenario
through `cargo dev record`, with the executable path given with the task; no committed file holds it. This change
completes the groundwork every command epic builds on.

## Change

- `scenarios/repo-remove/{removes-last-block,removes-middle-block,not-found,local-file-absent}/` exist, each with
  `scenario.toml` (`summary`, `[fixture] workspace_git = true`, one `[[step]]` with the table's arguments and
  `stdout = "json"`), `files/.winter/config.toml`, and the `record/` that `cargo dev record <id> --reference <path>`
  wrote. The configuration file is the spec's plain configuration in `removes-last-block` and its commented
  configuration in the other three, each byte for byte.
- `removes-middle-block` additionally has the `expected/` files and the `[deviation]` table of the spec's The deviation
  scenario. `expected/` is written by hand after the record exists: the configuration text from the spec, and the
  record's `after/tree.txt` with the one hash replaced. `record/` is not edited.
- The record of `removes-middle-block` is compared with the spec's prediction of the reference behaviour. When they
  differ, the change stops and reports it, so that the reference column of
  [repository-removal.md](../../../parallel-workspaces/repository-catalog/spec/repository-removal.md) is corrected
  first; the expectation is not adjusted to make replay pass.
- `crates/test-support/tests/proof.rs` gains the `repo-remove` rows of the spec's "Proof that a wrong implementation is
  caught".
- The reference tool's checkout is reached only through `cargo dev record`; no other command is run in it or on it. The
  recorder's own read-only git calls, before and after the run, are the whole contact
  ([record-and-replay.md](../spec/record-and-replay.md), "the reference checkout is read, never written").
- Only a record from a `cargo dev record` run that exited 0 is kept. A run that ended with the "reference checkout
  changed" refusal is reported with the paths it named, and its records are discarded.
- When a record shows the recorder, the runner, the snapshot, or the comparison to be at fault, the fault is fixed here
  together with a test that reproduces it with a stand-in executable. When replay of a scenario without a deviation
  fails because `repo remove` differs from the record, the command is corrected here, unless the correction would
  contradict repository-removal.md, What is removed: then the change stops and reports the record, as for the prediction
  above.
- Files touched: `scenarios/repo-remove/**`, `crates/test-support/tests/proof.rs`; and, only for a correction as above,
  the files of the scenario modules, of `crates/dev/src/record.rs`, or of `repo remove`.

## Acceptance

- `cargo scenarios repo-remove` reports four `ok` lines; `cargo dev gate full` exits 0 with
  `scenarios: 8 passed, 0 failed, 0 deferred`.
- The records can be made again, and making them leaves the reference checkout as it was. In a scratch copy of the
  checkout under a temporary directory, `cargo dev record --rerecord <the four ids> --reference <path>` exits 0 — so its
  closing check found the same `HEAD`, no tracked file modified, and the same set of untracked and ignored paths — and
  `git status` of the scratch copy then shows no change under `scenarios/`, a step's `stderr` file at most excepted,
  which replay never compares.
- Each record's `meta.toml` names the commit of `scenarios/reference.toml`; no file under `scenarios/` contains a
  temporary root path, the path of the reference executable, or the path of its checkout.
- `removes-last-block`: the scenario has no `expected/` directory and no `[deviation]` table; the binary's configuration
  file after the step is byte-equal to the record's and ends with the api url line followed by one blank line; the root
  repository's status block shows that one file modified and nothing else.
- `removes-middle-block`: `record/steps/01/after/files/ws/.winter/config.toml` has the line `# second: web` and lacks
  the line `# third: docs`; the `expected/` file has `# third: docs` and lacks `# second: web`, and differs from the
  record's file in that one line only; `scenario.toml` names the decision; replay compares the binary against
  `expected/` for the file and the tree and against `record/` for the exit code, the JSON document, and the git state,
  and passes.
- `not-found` and `local-file-absent`: the exit code is 1, `after/tree.txt` equals `before/tree.txt`, and no
  `.winter/config.local.toml` exists.
- `proof.rs`, with stand-in scripts on copies of the scenarios: on `removes-last-block`, one byte changed in the written
  file yields `Files` and `Tree` differences naming `ws/.winter/config.toml` and no other, and exact reproduction yields
  none; on `removes-middle-block`, writing the `expected/` file yields none, and writing the `record/` file yields
  `Files` and `Tree`.

## Out of scope

- `repo add` and `repo list`: `epic:parallel-workspaces/repository-catalog`; cloning:
  `epic:parallel-workspaces/workspace-bootstrap`; lock-file updates: `epic:parallel-workspaces/reproducible-inputs`.
- Capture of program calls: `program-capture`, which is not part of this groundwork.
