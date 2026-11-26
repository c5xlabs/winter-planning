---
name: process-group-calls
depends-on:
  - epic:parallel-workspaces/workspace-bootstrap
code:
  - winter-workspace/crates/core/src/programs.rs::Program
  - winter-workspace/crates/core/src/programs.rs::Call
  - winter-workspace/crates/core/src/programs.rs::OsPrograms
---

# Argument-vector programs in their own process group

Extends the program port as [value-lookups.md](../spec/value-lookups.md), Domain and The child, give it:
`Program::Argv`, whose first argument is the program; `Group` and `Call.group`, where `Group::Own` starts the child as
the leader of a new process group and a timeout kills the whole group; and `Call.interrupts`, with which the call defers
the interrupt for its duration and kills the group once one is pending. It builds on the `Program::Shell`, `Stdin`, and
`remove` that `epic:parallel-workspaces/workspace-bootstrap` adds. Delivers the process-group and timeout mechanism of
`engineering:bounded-value-commands`, under `decision:running-environments/scope-environment/lookup-execution-policy`,
whose group kill, on the deadline and on an interrupt, is a registered deviation. The deferral is that deviation's
means, which value-lookups.md, The child, adds beside the service paths of the frozen
[runtime.md](../../../../delivered/parallel-workspaces/component-runtime/spec/runtime.md), Interrupts and signals,
without editing it; what a user sees of an interrupt stays the reference's.

## Change

- `Program::Argv` runs `args[0]`: a word holding `/` is that path, relative to the call's `cwd`, which a lookup sets to
  the root; any other is looked up on the child's `PATH` as `find_program` does.
- `Call` gains `group: Group` and `interrupts: Option<&Interrupts>`, `Some` only with `Group::Own`; every existing call
  passes `Group::Inherit` and `None` and behaves as before. `Group` is re-exported on the `programs` line of `lib.rs` as
  `ProcessGroup`, since `fan_out::Group` of `epic:parallel-workspaces/coordinated-git` holds `Group` at the crate root.
- With `Group::Own`, `OsPrograms` puts the child in its own group before exec; on the timeout it sends `SIGKILL` to the
  group and reaps the child, ending as the existing timeout does.
- With `interrupts`, the call defers before the start and releases after the reap; it waits in slices of at most 100 ms,
  and once `pending()` is true it kills the group, reaps, and returns; the caller reads `pending()` after the release,
  as runtime.md asks of every command that defers.
- `context/architecture/changes.md`, A command, gains that a program-port call given `interrupts` defers and releases
  around its child on the command's behalf, which is how a command defers around a child in its own process group, and
  that the command still reads `pending()` after the call and ends the invocation when it is true.

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
- A component test: with `interrupts` and a child sleeping 60 s, `note()` from another thread ends the call within 200
  ms, the group is gone, `pending()` is true, and `is_deferred()` is false after the call.
- Every existing test passes.
- `cargo dev gate slice` exits 0.
