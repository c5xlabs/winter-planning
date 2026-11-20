---
name: proof-read-only
depends-on:
  - replay-target
  - record-command
  - index-command
---

# Proof on a read-only behaviour: ws index recorded and replayed

Records the four `ws-index` scenarios of [method-proof.md](../spec/method-proof.md), Proof scenarios, against the
reference tool, replays them with the binary, and proves a wrong implementation is caught. Delivers the read-only half
of `engineering:method-proof`. This is the first change in which the reference tool runs: once per scenario, through
`cargo dev record`, by the person or agent making the change. The path of the reference executable is given to them with
the task; no committed file holds it.

## Change

- `scenarios/ws-index/{alias-suggested,hashed-suggested,registry-recorded,suggested-slot-taken}/` exist, each with
  `scenario.toml` (a `summary`, one `[[step]]` with the table's arguments and `stdout = "json"`, no `[fixture]` table,
  no `needs`), the `files/` the table describes, and the `record/` that `cargo dev record <id> --reference <path>`
  wrote. The `.winter/state.toml` of `registry-recorded` and of `suggested-slot-taken` has the form of
  [lock-and-state.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/lock-and-state.md),
  `.winter/state.toml`; the latter's content is given whole in the method-proof spec.
- `crates/test-support/tests/proof.rs` copies `scenarios/ws-index/hashed-suggested` and
  `scenarios/ws-index/suggested-slot-taken` into a temporary scenarios root and replays them with stand-in scripts,
  asserting the `ws-index` rows of the spec's "Proof that a wrong implementation is caught".
- The reference tool's checkout is reached only through `cargo dev record`. No other command is run in it or on it,
  `git status` included: the recorder's own read-only git calls, before and after the run, are the whole contact
  ([record-and-replay.md](../spec/record-and-replay.md), "the reference checkout is read, never written").
- Only a record from a `cargo dev record` run that exited 0 is kept. A run that ended with the "reference checkout
  changed" refusal is reported with the paths it named, and its records are discarded.
- When a record shows the recorder, the runner, the snapshot, or the comparison to be at fault, the fault is fixed here
  together with a test that reproduces it with a stand-in executable.
- When replay fails because `ws index` differs from the record, the command is corrected here. When the record
  contradicts index-lookup.md or a section it links, the change stops and reports the contradiction; neither the record
  nor the scenario is edited to hide it.
- Files touched: `scenarios/ws-index/**`, `crates/test-support/tests/proof.rs`; and, only for a correction as above, the
  files of the scenario modules, of `crates/dev/src/record.rs`, or of `ws index`.

## Acceptance

- `cargo scenarios ws-index` reports four `ok` lines and `scenarios: 4 passed, 0 failed, 0 deferred`;
  `cargo dev gate full` exits 0.
- The records can be made again, and making them leaves the reference checkout as it was. In a scratch copy of the
  checkout under a temporary directory, `cargo dev record --rerecord <the four ids> --reference <path>` exits 0 — so its
  closing check found the same `HEAD`, no tracked file modified, and the same set of untracked and ignored paths — and
  `git status` of the scratch copy then shows no change under `scenarios/`, a step's `stderr` file at most excepted,
  which replay never compares.
- Each record's `meta.toml` names the commit of `scenarios/reference.toml`; no file under `scenarios/` contains a
  temporary root path, the path of the reference executable, or the path of its checkout.
- In each record, `after/tree.txt` of step 01 equals `before/tree.txt`, and `after/files/` is empty.
- The recorded `stdout.json` documents have `source` `suggested`, `suggested`, `registry`, and `suggested` in the
  table's order, and the `index` of `hashed-suggested` is the `feature-x` value of lock-and-state.md, Fixed points.
- The recorded `index` of `suggested-slot-taken` is 40, the value its own `files/.winter/state.toml` records for
  `other`. A record with another value contradicts index-lookup.md and is reported as the Change section says.
- `proof.rs`: on `hashed-suggested`, the stand-in with another exit code yields exactly one `Exit` difference; with the
  `index` value changed, one `Json` difference whose subject is `/index`; reproducing the record, none. On
  `suggested-slot-taken`, the stand-in printing `index` 41 yields one `Json` difference whose subject is `/index`.

## Out of scope

- The state-changing proof: `proof-state-changing`.
- Human output fidelity of `ws index`: `epic:parallel-workspaces/environment-lifecycle`.
