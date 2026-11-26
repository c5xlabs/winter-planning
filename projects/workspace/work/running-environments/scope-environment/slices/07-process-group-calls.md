---
name: process-group-calls
depends-on:
  - epic:parallel-workspaces/workspace-bootstrap
code:
  - winter-workspace/crates/core/src/programs.rs::Program
  - winter-workspace/crates/core/src/programs.rs::Call
  - winter-workspace/crates/core/src/programs.rs::RunPrograms
  - winter-workspace/crates/core/src/programs.rs::OsPrograms
  - winter-workspace/crates/core/src/process.rs::Invocation
---

# Argument-vector programs and the process group of a call

Extends the program port as [value-lookups.md](../spec/value-lookups.md), Domain and The child, give it:
`Program::Argv`, whose first argument is the program; `Group` and `Call.group`, where `Group::Own` starts the child as
the leader of a new process group and a timeout kills the whole group, and `Group::Shared` keeps the child in
Workspace's process group and a timeout kills the direct child alone; `RunPrograms::has_controlling_terminal`, the fact
a lookup's group is chosen by, which the binary detects and carries on `Invocation`; and `Call.interrupts`, with which
the call defers the interrupt for its duration and kills the group or the child once one is pending. It builds on the
`Program::Shell`, `Stdin`, and `remove` that `epic:parallel-workspaces/workspace-bootstrap` adds. Delivers the
process-group and timeout mechanism of `engineering:bounded-value-commands`, under
`decision:running-environments/scope-environment/lookup-execution-policy`, whose group kill without a controlling
terminal, on the deadline and on an interrupt, is a registered deviation, and whose child kill with one is the
reference's. The deferral is the means of either kill, which value-lookups.md, The child, adds beside the service paths
of the frozen [runtime.md](../../../../delivered/parallel-workspaces/component-runtime/spec/runtime.md), Interrupts and
signals, without editing it; what a user sees of an interrupt stays the reference's.

## Change

- `Program::Argv` runs `args[0]`: a word holding `/` is that path, relative to the call's `cwd`, which a lookup sets to
  the root; any other is looked up on the child's `PATH` as `find_program` does.
- `Call` gains `group: Group` and `interrupts: Option<&Interrupts>`, `Some` only with `Group::Own` or `Group::Shared`;
  every existing call passes `Group::Inherit` and `None` and behaves as before. `Group` is re-exported on the `programs`
  line of `lib.rs` as `ProcessGroup`, since `fan_out::Group` of `epic:parallel-workspaces/coordinated-git` holds `Group`
  at the crate root.
- With `Group::Own`, `OsPrograms` puts the child in its own group before exec; on the timeout it sends `SIGKILL` to the
  group and reaps the child, ending as the existing timeout does.
- With `Group::Shared`, `OsPrograms` leaves the child in Workspace's process group; on the timeout it sends `SIGKILL` to
  the direct child alone and reaps it, ending as the existing timeout does, and kills none of the child's descendants.
- Group selection: `RunPrograms` gains `has_controlling_terminal`, `false` unless a port answers otherwise;
  `OsPrograms::with_controlling_terminal` sets what it answers. `Invocation` gains `has_controlling_terminal`, which the
  binary sets once, beside `stdout_is_terminal`, by whether opening `/dev/tty` with `O_RDWR | O_NOCTTY` succeeds, and
  which `process.rs` hands the invocation's `OsPrograms`; `InProcess` passes `false` unless a test sets it. A test
  injects the fact through the program port, or through `InProcess`, so neither case needs a real terminal.
- With `interrupts`, the call defers before the start and releases after the reap; it waits in slices of at most 100 ms,
  and once `pending()` is true it kills the group (`Group::Own`) or the direct child (`Group::Shared`), reaps, and
  returns; the caller reads `pending()` after the release, as runtime.md asks of every command that defers.
- `context/architecture/changes.md`, A command, gains that a program-port call given `interrupts` defers and releases
  around its child on the command's behalf, which is how a command defers around a lookup's child, and that the command
  still reads `pending()` after the call and ends the invocation when it is true.

## Acceptance

- Component tests in `crates/core/tests/programs.rs`: `Program::Argv` runs a program found on the given `PATH` with its
  arguments as given, a word holding a space included; a relative `./bin/x` runs from the call's `cwd`; an absent
  program is the `io::Error` of a failed start.
- A component test: a `Group::Own` shell child that starts `sleep 60 &` and waits, under a 1 s timeout, ends as the
  timeout, and afterwards no process of its group is alive.
- A component test: a `Group::Own` shell child that prints its own process id and process group id prints the same
  number twice, and not the test process's group, so a terminal's `SIGINT` to the foreground group does not reach it;
  the test sends no signal
  ([test-harness.md](../../../../delivered/parallel-workspaces/component-runtime/spec/test-harness.md), `invoke`,
  signals sent).
- A component test: a `Group::Shared` shell child that starts `sleep 60 &`, prints the sleep's process id, and waits,
  under a 1 s timeout, ends as the timeout, the shell is gone, and the printed sleep is still alive, which the test then
  kills.
- A component test: a `Group::Shared` shell child prints the test process's process group id as its own.
- Component tests: with `interrupts` and a child sleeping 60 s, `note()` from another thread ends the call within 200
  ms, `pending()` is true, and `is_deferred()` is false after the call; under `Group::Own` the group is gone, and under
  `Group::Shared` the child is.
- Unit tests: `OsPrograms` answers `has_controlling_terminal` as `with_controlling_terminal` set it, `false` when unset;
  a port that does not override it answers `false`; the `OsPrograms` that `process.rs` builds for an invocation answers
  the invocation's `has_controlling_terminal`, given `true` and given `false`.
- Every existing test passes.
- `cargo dev gate slice` exits 0.
