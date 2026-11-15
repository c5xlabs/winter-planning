---
name: program-capture
depends-on:
  - replay-target
  - scenario-context
---

# Stub programs and the calls dimension

Builds `scenario::stub` and the `Calls` comparison: the `[[stub]]` table and Program calls section of
[scenario-format.md](../spec/scenario-format.md) and the `Calls` row of
[record-and-replay.md](../spec/record-and-replay.md). Delivers the hand-off clauses of
`engineering:behavioral-fixtures`. It is not part of the groundwork that `engineering:method-proof` completes: no proof
scenario uses a stub, and this change is built before the first epic whose behaviour starts another program, not before
the epics that start none.

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
- A scenario without `[[stub]]` has no `calls/` directory in its record, and the eight proof scenarios replay unchanged.
- `cargo dev gate full` exits 0.

## Out of scope

- Stubs that answer differently per call: the open question of the scenario format spec.
- Live providers: the `provider` facility.
