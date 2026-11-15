---
name: diagnostics-and-logging
depends-on:
  - root-options
code:
  - winter-workspace/crates/core/src/process.rs::run
  - winter-workspace/crates/core/src/logging.rs
---

# Log records on stderr

Builds `diagnostics::record` of [runtime.md](../spec/runtime.md), Diagnostics on the stderr handle, and the record
formats of [output-formats.md](../spec/output-formats.md), Diagnostics, under `engineering:machine-output-boundary`,
`decision:parallel-workspaces/component-runtime/unswitched-log-records`, and
`decision:parallel-workspaces/component-runtime/log-level-names`. The runtime writes its one record, which gives the
writer its first caller. It follows `root-options` and is built before `process-boundary`; it runs beside no other
change.

## Change

- `crates/core/src/diagnostics.rs` exists with `Record` and `record` as the spec gives them: the threshold test, the
  switched and unswitched formats, `only_when_switched`, and a failed write to the handle ignored.
- `process.rs`: step 2 of `run_with` ([runtime.md](../spec/runtime.md), The parse and the run) writes the one Debug
  record the spec names, carrying the command path, to the stderr handle of the `Invocation`, at `Stage::Command`
  whatever the outcome. `parse.rs` is not touched: the parser writes nothing, and the record needs no `Context`.
- Contributor documents: `context/architecture/structure.md` gains the `diagnostics.rs` row;
  `context/standards/rust.md`, Output and environment access, says a module reports a condition through
  `diagnostics::record` and never formats a record itself.
- Files touched: `crates/core/src/diagnostics.rs` (new), `crates/core/src/{process,lib}.rs`,
  `crates/core/tests/root.rs`, `context/architecture/structure.md`, `context/standards/rust.md`, and the scenario
  directories below. No manifest changes.
- `crates/dev/tests/architecture.rs` pins nothing this change alters; `diagnostics.rs` writes only to the handle it is
  given. `diagnostics::notice` is not written: nothing calls it yet, and the gate denies unused code.
- Scenarios recorded with `cargo dev record`, both with `stdout = "json"`: `ws-index/verbose`
  (`-v ws index NAME
  --json`) and `ws-index/log-level-env` (the step's environment sets `WINTER_LOG_LEVEL=DEBUG`).
  They hold that logging never reaches stdout; component tests hold what stderr carries, which no scenario compares.

## Acceptance

- Unit tests of `diagnostics.rs`, each with a handle the test owns:
  - a record below the threshold writes nothing;
  - a switched log writes one line holding the level name, the origin, and the message, in the order the spec gives;
  - an unswitched log writes a Warning record as its message alone and drops an Info record;
  - `Log { threshold: Level::Warning, switched: true }`, which `NOTSET` gives, drops a Debug record and writes a Warning
    record in the switched format;
  - `only_when_switched` writes nothing under an unswitched log at any level;
  - a handle whose writes fail makes `record` return without a panic.
- Component tests, through the scaffold: `-v ws index NAME --json` leaves stdout one JSON line and stderr holding the
  record with the command path; the same without `-v` leaves stderr empty; `WINTER_LOG_LEVEL=debug` without `-v` writes
  the record; `WINTER_LOG_LEVEL=bogus` and `WINTER_LOG_LEVEL=NOTSET` write nothing and exit 0; `-v --help` and
  `-v --version` write no record; `-v ws --help` and `-v ws index --bogus` write the record, naming `ws` and `ws index`.
- The two scenarios pass, and `cargo dev gate full` exits 0.

## Out of scope

- `diagnostics::notice`: built with the first command that writes one.
- The git-failure record of `decision:parallel-workspaces/component-runtime/git-failure-rendering`: the epic whose
  command first calls git.
- The two records about tracing: `command-span-export`.
