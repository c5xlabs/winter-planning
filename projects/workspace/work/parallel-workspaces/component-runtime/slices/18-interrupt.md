---
name: interrupt
depends-on:
  - process-boundary
code:
  - winter-workspace/crates/cli/src/main.rs::main
  - winter-workspace/crates/core/src/process.rs::Invocation
  - winter-workspace/crates/core/src/context.rs::Context
---

# The interrupt

Builds `Interrupts` and the SIGINT handler of [runtime.md](../spec/runtime.md), Interrupts and signals, under
`engineering:handoff-fidelity`, `decision:parallel-workspaces/component-runtime/signal-disposition`, and
`decision:parallel-workspaces/component-runtime/interrupt-trace-export`. It follows `process-boundary` and needs no
other change; `stdout-fault-tests` needs the same one and may be built beside this change, and `runtime-documents`
follows both.

## Change

- `crates/core/src/process.rs`: `Interrupts` exists with the methods the spec gives; `Invocation` and `Context` carry
  it, and `run_in_memory` passes one of its own. `Termination::Aborted` and `Termination::Interrupted` are not built:
  the handler ends the process itself, and no command of the tree returns either.
- `crates/cli/src/interrupt.rs` exists: it installs the handler, which does only what the spec lists for the deferred
  and the undeferred case, and each `unsafe` block states why it is sound. `main` installs it before `run`.
- `crates/cli/Cargo.toml` names `libc` 0.2, and `Cargo.lock` gains the binary's edge to it. The crate is already
  resolved in the lock for the support and driver packages, so nothing is fetched.
- The scaffold's `InProcess` passes an `Interrupts` of its own and gains `interrupt_pending`.
- The handler's tests need no change to the support package: the child raises the signal on itself, as the spec's
  "testing the handler" gives it, and `invoke` neither sends a signal nor exposes a process id.
- Contributor documents, each a sentence this change makes false or a rule it adds:
  - `context/architecture/structure.md`: the sentence "`crates/cli/src/main.rs` … is the only file that names the
    process streams, the process environment, or the process exit" names the binary package's two files and what each
    names; the `interrupt.rs` row is added. No dependency table exists in the document yet, and this change adds none:
    `runtime-documents` writes it from the manifests.
  - `context/architecture/changes.md`: a command defers the interrupt only around a child it must outlive.
  - `context/standards/testing.md`: the Tiers sentence "A unit test takes no file, git, or process input and uses
    nothing from the support package" gains the exception of [test-harness.md](../spec/test-harness.md), the unit tier's
    exceptions, for the binary package; "Tests that must fail" says its child-test pattern is the one the binary
    package's handler tests use. Regions shared with `stdout-fault-tests`: these are the only regions of the file this
    change edits; that change edits the item of the Tiers list that begins "An executable test starts the binary", and
    whichever of the two merges second rebases.
  - `context/standards/rust.md`: `unsafe` appears in `crates/cli/src/interrupt.rs` alone, each block with its reason.
  - `AGENTS.md`, the library-boundary rule, names the binary package where it named `crates/cli/src/main.rs` alone, as
    [contributor-context.md](../spec/contributor-context.md), Entry point, words it.
- Files touched: `crates/cli/src/interrupt.rs` (new), `crates/cli/src/main.rs`, `crates/cli/Cargo.toml`, `Cargo.lock`,
  `crates/core/src/{process,context,lib}.rs`, `crates/core/tests/support/in_process.rs`, `crates/core/tests/root.rs`,
  `context/architecture/{structure,changes}.md`, `context/standards/{testing,rust}.md`, `AGENTS.md`.
  `crates/core/src/termination.rs` and `crates/test-support/` are not touched.
- `crates/dev/tests/architecture.rs`: the package-edge rules read workspace edges only, so a third-party dependency of
  the binary alters no pin; the change confirms it by running the test. The binary package gains its first dependency on
  the support package's use from a unit test, which is a development dependency the manifest already holds for its
  executable tests. The scope-token scan reads `AGENTS.md`, and the reworded rule holds none of the tokens. The
  dependency table of [repository-layout.md](../spec/repository-layout.md), Cargo workspace, already gives the binary
  `libc`; the open question there on the key order of a manifest is not raised by adding one dependency line.
- Scenarios: none; no present command runs long enough to interrupt, and the scenario format sends no signal.

## Acceptance

- Unit tests of `Interrupts`: `defer` and `release` nest; `note` then `pending` is true; `pending` is false on a fresh
  value.
- A component test, through the scaffold: `interrupt_pending` leaves `ws index NAME` ending as it does without it, since
  no present command defers.
- Unit tests in `crates/cli/src/interrupt.rs`, each starting a child test of the same test binary through the support
  package's `invoke`, by the fixture rule of the unit tier's exceptions; the child installs the handler, writes a ready
  line to stdout, and raises SIGINT on itself:
  - not deferred: the child exits 1, its stderr holds the aborted notice, and its stdout holds the ready line and
    nothing after it;
  - deferred: the handler returns, the child sees `pending` true, writes a second line, and exits 0.
- `grep -rn 'unsafe' crates/cli/src` prints lines of `interrupt.rs` only, each block preceded by a comment.
- `grep -rnE 'Termination::(Aborted|Interrupted)' crates/core/src` prints nothing, and the enum declares neither
  variant: `grep -nE '^[[:space:]]*(Aborted|Interrupted)' crates/core/src/termination.rs` prints nothing.
- `cargo fetch --locked` exits 0 with no network.
- `cargo dev gate full` exits 0.

## Out of scope

- Any caller of `defer`, and the exit of a service command interrupted while its provider runs:
  `epic:running-environments/service-orchestration`.
- `Termination::Interrupted`, and `Termination::Aborted` from a declined prompt: each is built with the first command
  that returns it.
- That SIGTERM ends the process by the signal's default disposition: the binary installs nothing for it, and no test of
  this change sends one.
- That an interrupted run sends no span: `command-span-export` holds it once a span exists.
