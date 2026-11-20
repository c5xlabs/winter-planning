---
name: program-capture
depends-on:
  - epic:rust-adoption/parity-validation
code:
  - winter-workspace/crates/test-support/src/scenario/fixture.rs::Plan::of
  - winter-workspace/crates/test-support/src/scenario/runner.rs::CALL_CAPTURE
  - winter-workspace/crates/test-support/src/scenario/compare.rs::Dimension
---

# Stub programs and the calls dimension

Builds `scenario::stub` and the `Calls` comparison: the `[[stub]]` table and Program calls of
[scenario-format.md](../../../../delivered/rust-adoption/parity-validation/spec/scenario-format.md) and the `Calls` row
of [record-and-replay.md](../../../../delivered/rust-adoption/parity-validation/spec/record-and-replay.md). Delivers
`engineering:program-call-capture` under `decision:rust-adoption/parity-validation/program-capture-placement`. Lands
before the first command of this epic that starts a program, so that command's recorded scenarios can stub it.

## Change

- The fixture builder writes one recording program per `[[stub]]` entry into `<ROOT>/bin` and no longer refuses a
  scenario that has one; `runner::CALL_CAPTURE` becomes `true`, and the `stub` refusals of record and replay are removed
  with their tests.
- Each stub invocation writes one numbered call record under `<ROOT>/calls/` and then prints and exits as declared.
- The step runner collects the call records of each step into `calls/`, root-normalized.
- Comparison yields `Calls` differences for a different call count, program, argument vector, working directory, or
  listed environment variable; the record step's stability check covers them through the same function.
- `context/verification/scenarios.md` gains how to declare a stub and read a `Calls` difference.
- Files touched: `crates/test-support/src/scenario.rs`,
  `crates/test-support/src/scenario/{stub,fixture,runner,record,compare}.rs`,
  `crates/test-support/tests/{scenario_calls,scenario_fixture,scenario_replay}.rs`, `crates/dev/src/record.rs`,
  `crates/dev/tests/record.rs`, `context/verification/scenarios.md`.

## Acceptance

- A scenario stubbing `docker`, run with a stand-in that calls `docker compose up -d` twice from the workspace
  directory, records two call files in start order with the argument vector, `<ROOT>/ws` as working directory, and
  exactly the listed variables.
- An argument containing a space, a quote, and a newline round-trips into `argv` unchanged.
- A listed variable that is unset is absent from `env`; an unlisted one that is set is absent too.
- Replay with a stand-in that passes one different argument yields one `Calls` difference naming the call number; one
  call fewer yields a count difference.
- A stub declared with `exit = 3` makes the calling stand-in observe 3.
- A scenario without `[[stub]]` has no `calls/` directory in its record, and every recorded scenario replays unchanged.
- `cargo dev gate full` exits 0.

## Out of scope

- Stubs that answer differently per call: the open question of the scenario format spec.
- Live providers: the `provider` facility.
