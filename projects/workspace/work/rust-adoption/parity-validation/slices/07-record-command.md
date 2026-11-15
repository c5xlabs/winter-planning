---
name: record-command
depends-on:
  - replay-target
---

# The record command and its refusals

Adds `cargo dev record` to the development driver: [record-and-replay.md](../spec/record-and-replay.md), System and
Record. Delivers `engineering:record-step-separation` under
`decision:rust-adoption/parity-validation/recorder-placement`. This is the only code that is handed the reference tool's
path, and no test in this change starts the reference tool: every test uses a stand-in executable inside a stand-in git
checkout the test creates under a temporary directory.

## Change

- `cargo dev record <id>…` and `cargo dev record --all-missing` resolve the reference executable from `--reference` or
  `WINTER_REFERENCE_BIN`, apply every refusal of the spec before starting it, and call `scenario::record` with the
  reference-only environment additions, the one redaction of the spec, and the `reference_runtime` line.
- The command reads `WINTER_REFERENCE_BIN` and `PATH` through the environment lookup `main_with` is given, as the spec's
  Record block and [gates.md](../../../parallel-workspaces/component-runtime/spec/gates.md), Steps, say; the driver's
  `main.rs` already passes that lookup and does not change.
- The refusals are one function of the driver's library over explicit inputs — the reference path, the caller's `PATH`
  value, the scenarios root — so a test passes each input and no test changes its own process environment.
- The checkout is inspected with exactly the three git commands and the environment of the spec's "the reference
  checkout is read, never written": one pass before the reference is first started and one after the last scenario of
  the run.
- The closing check of the spec compares the two passes. A changed checkout ends the run with exit 1 and the spec's
  report of added, removed, and modified paths; an untracked or ignored file that was already there before the run is
  not a change.
- A scenario whose `needs` the runner cannot satisfy is skipped with a line saying so.
- Exit codes are 0, 1, and 2 as specified.
- The architecture test gains the two substring rules of the spec's System section, each over the files that section
  gives it. Those files already satisfy both, as
  [test-harness.md](../../../parallel-workspaces/component-runtime/spec/test-harness.md), "names the package never
  spells", required of every change that wrote them. A violation found there is reported with its file and line, and the
  offending text is removed in this change by rewording a comment or renaming a test's own variable; no behaviour of
  those packages changes.
- `AGENTS.md` gains the standing rule about the reference tool from
  [contributor-context](../../../parallel-workspaces/component-runtime/spec/contributor-context.md), Entry point.
- Files touched: `crates/dev/src/{lib,record,architecture}.rs`, `crates/dev/tests/{record,architecture}.rs`,
  `crates/dev/Cargo.toml`, `Cargo.lock`, `AGENTS.md`; and, only for a violation as above, the one file that holds it.

## Acceptance

- With neither `--reference` nor the variable, the command exits 2 and creates no temporary tree.
- One test per refusal asserts exit 1, a message naming what was found, and that the stand-in was never started (it
  would create a marker file): a relative path; a missing file; a non-executable file; an executable that is the first
  `winter` on the given `PATH`, reached directly and through a symbolic link; an executable with no `.git` entry above
  it; a checkout under a directory whose name holds a space; a checkout whose `HEAD` differs from
  `scenarios/reference.toml`; a checkout with a modified tracked file; an existing `record/` without `--rerecord`; a
  scenario with a `[[stub]]`.
- The same holds for a scenario that is not confined, one test per case, each with a stand-in in a checkout at the
  pinned commit, so that only the scenario is at fault: a step `cwd` of `/tmp`; a step `cwd` of `../..`; a checkout
  `path` of `../outside`. Each exits 1 with a message naming the scenario id and the key, the marker file does not exist
  — the stand-in was not started, its `--version` call included — and no `record/` and no temporary tree was created.
- With a stand-in executable in a checkout at the pinned commit, recording one scenario exits 0 and writes `record/`
  whose `meta.toml` names that commit and the stand-in's `--version` line.
- A stand-in that prints the path of its own checkout on stderr records `<REFERENCE>` in `stderr`, and no file under the
  written `record/` contains that path.
- In that test the stand-in checkout's `.git/index` has the same bytes and modification time after the record as before,
  also when a tracked file's modification time was changed without changing its content beforehand; and no file was
  created anywhere in the checkout.
- The closing check, each case with a stand-in checkout at the pinned commit:
  - the checkout holds an untracked file and a git-ignored directory with a file in it before the run, and the stand-in
    writes nothing there: exit 0;
  - the stand-in creates an untracked file in its checkout: exit 1, and stderr has the "reference checkout changed" line
    and that path with `+`;
  - the stand-in creates a file inside the git-ignored directory: exit 1, naming that path with `+`;
  - the stand-in deletes the untracked file that was there before: exit 1, naming it with `-`;
  - the stand-in appends to a tracked file: exit 1, naming it with `M`.
- A run refused before the stand-in was started runs each of the three git commands once; a run that started it runs
  each twice. The test counts the calls through a recording `git` placed first on the `PATH` it passes in.
- `--all-missing` records the scenario without a record and leaves a recorded one byte-identical;
  `--all-missing --rerecord` exits 2.
- The stand-in observes the two interpreter variables of the spec with the values 1 and 0; the recorded environment
  digest equals the digest without them.
- For the variable rule a unit test feeds a violating file under `crates/core`, under `crates/cli`, and under
  `crates/test-support`; for the word rule, one under `crates/test-support` and a violating
  `crates/cli/tests/scenarios.rs`. Each asserts the failure names the file. A file under `crates/core/src` and one under
  `crates/cli/src` holding the word are not reported. The architecture test passes on the checkout.
- `cargo dev gate full` exits 0.

## Out of scope

- Recording any real scenario: `proof-read-only`.
