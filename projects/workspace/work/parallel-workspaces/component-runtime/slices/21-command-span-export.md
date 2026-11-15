---
name: command-span-export
depends-on:
  - command-span-record
code:
  - winter-workspace/crates/cli/src/main.rs::conclude
  - winter-workspace/crates/cli/Cargo.toml
  - winter-workspace/crates/core/src/process.rs::Invocation
  - winter-workspace/crates/core/src/trace.rs
---

# The command span is opened, ended, and exported once

Builds the tracer port, the two records, The binary's part, and Flows of [tracing.md](../spec/tracing.md), with the
fourth parameter of `conclude` and `Invocation.tracer` of [runtime.md](../spec/runtime.md), under
`engineering:observability-isolation`, `decision:observability-shutdown`,
`decision:parallel-workspaces/component-runtime/trace-export-transport`,
`decision:parallel-workspaces/component-runtime/trace-exporter`,
`decision:parallel-workspaces/component-runtime/interrupt-trace-export`, and
`decision:parallel-workspaces/component-runtime/trace-export-trust-roots`: the exporter verifies an `https` endpoint
against the public roots bundled in the binary and reads no other trust input. It follows `command-span-record`, is the
last change of the epic, and runs beside no other change.

## Change

- `crates/cli/Cargo.toml` names `opentelemetry`, `opentelemetry_sdk`, and `opentelemetry-otlp`, each at the newest
  release that builds on toolchain 1.99.1, with default features off and the features [tracing.md](../spec/tracing.md),
  The binary's part, lists. The TLS feature taken is the one with bundled public roots, and none that reads a system
  store, under `decision:parallel-workspaces/component-runtime/trace-export-trust-roots`. `Cargo.lock` gains the three
  crates and everything they pull in. The build of this change fetches from crates.io; after it, `cargo fetch --locked`
  is again the only step that needs the network. The change confirms each feature name, and each point the spec's Open
  questions mark as remembered, against the releases it takes; one that does not hold and changes anything the spec
  lists under Function goes back to engineering before the change merges.
- `crates/core/src/trace.rs`: `CommandTracer`, `NoTrace`, and `TraceLog` exist as the spec gives them; the two records
  go through `diagnostics::record`.
- `crates/core/src/process.rs`: `Invocation` carries `tracer`; step 2 of `run_with` announces the command once, after
  the Debug record, at `Stage::Command` whatever the outcome; `run_in_memory` passes `NoTrace`. `ProcessEnv::names()`
  exists as [repository-layout.md](../spec/repository-layout.md) gives it.
- `crates/cli/src/trace.rs` exists with `Tracer`, `Tracer::new`, `Tracer::with_exporter`, `finish`, and
  `hide_generic_exporter_variables`. Its one `unsafe` block is the removal of the variables, with the reason the spec
  gives.
- `crates/cli/src/main.rs`: `main` reads `settings`, hides the generic exporter variables when they are `On`, builds the
  tracer, and passes it; `conclude` has its fourth parameter and step 5.
- The scaffold passes a recording tracer, and `Ended` gains `announced`.
- Contributor documents, each a sentence this change makes false or a rule it adds:
  - `AGENTS.md`, the library-boundary rule, names sockets, as [contributor-context.md](../spec/contributor-context.md),
    Entry point, words it.
  - `context/architecture/structure.md`: the Modules table gains the row of the binary's `trace.rs` and the library's
    row names the port; the sentence on which files of the binary package name the process environment names `trace.rs`;
    the dependency table gains the three crates, each with its features and its user.
  - `context/standards/rust.md`: `unsafe` appears in `crates/cli/src/interrupt.rs` and `crates/cli/src/trace.rs` alone;
    Output and environment access says the library opens no socket, reads no clock, and names no tracing crate.
  - `context/standards/testing.md`: a test wanting a span clears the kill switch of the pinned environment; the unit
    tier's exception for the binary package covers the collector in the tests of `crates/cli/src/trace.rs`.
  - `context/tooling/development.md`: the prerequisites name a C compiler if the TLS stack's build needs one; the
    sentence that names bootstrap as the only step that needs the network stays.
- The backlog's dependency table, [repository-layout.md](../spec/repository-layout.md), The Cargo workspace, receives
  the versions and feature names this change took.
- Files touched: `crates/cli/Cargo.toml`, `Cargo.lock`, `crates/cli/src/trace.rs` (new), `crates/cli/src/main.rs`,
  `crates/core/src/{trace,process,lib}.rs`, `crates/core/tests/support/in_process.rs`, `crates/core/tests/root.rs`,
  `crates/cli/tests/trace.rs` (new), `AGENTS.md`, `context/architecture/structure.md`,
  `context/standards/{rust,testing}.md`, `context/tooling/development.md`. `crates/test-support/` is not touched: the
  collector exists.
- `crates/dev/tests/architecture.rs`: the package-edge rules read workspace edges only, so third-party dependencies of
  the binary alter no pin; the change confirms it by running the test. The library's `trace.rs` and `process.rs` spell
  none of the forbidden tokens; `AGENTS.md` and the context documents gain no scope token and no command;
  `crates/cli/tests/trace.rs` holds neither the reference variable's name nor the word the support package never spells.
- Scenarios: none ([tracing.md](../spec/tracing.md), Test seams).

## Acceptance

- Unit tests in `crates/cli/src/main.rs`, each calling `conclude` with closures of its own:
  - `finish` is called once, after the diagnostic is written, with the `error_type` of the final ending: `None` after
    `Success`, `Some("Usage")` after a usage ending, `Some("BrokenPipe")` when the flush fails with that kind after
    `Success`, `Some("Internal")` after a panic;
  - a `finish` that panics, and one that writes to a failing stderr, leave the exit code as the ending gives it.
- Unit tests in `crates/cli/src/trace.rs`, through `Tracer::with_exporter` with an exporter of the test's own that
  records the span data it is handed, and no socket:
  - settings `Off`: the builder is never called, and `finish` writes nothing, announced or not;
  - settings `On` and no command announced: the builder is never called;
  - settings `On` and a command announced: the exporter is handed one span, with the name, kind internal, the one
    attribute `winter.command`, the launch time as its start or an instant not before the test began when there is none,
    and the resource of the settings; with an error type, exactly the two attributes and an error status with an empty
    message;
  - a sampled `TRACEPARENT`: the span has its trace id and its span id as parent; an invalid one: a root span of a new
    trace; an unsampled one: the exporter is handed nothing and `finish` writes nothing;
  - an exporter that answers an error: nothing on stderr under an unswitched log, one record of origin `trace` under a
    switched one;
  - a builder that answers an error, and settings `Unavailable`: nothing is exported; one record under a switched log at
    Debug, none otherwise, and none when no command was announced.
- Unit tests in `crates/cli/src/trace.rs`, `Tracer::new` against a `Collector` in the test process:
  - `Answer::Ok`: one request arrives with method `POST`, a target ending in `/v1/traces`,
    `content-type: application/x-protobuf`, and the configured headers; its body read through `decode_export` holds the
    one span; `finish` writes nothing under a switched log;
  - `Answer::Status(500)`: one request arrived, and `finish` writes the export record under a switched log;
  - `Answer::Hang`: `finish` returns within 1 s and writes the export record;
  - `Collector::refused_endpoint()`: `finish` writes the export record.
- A unit test of `hide_generic_exporter_variables`, in a child test of the same test binary by the fixture rule of the
  unit tier's exceptions: with `OTEL_EXPORTER_OTLP_ENDPOINT` and `OTEL_EXPORTER_OTLP_HEADERS` in the child's
  environment, after the call the process environment holds neither and the snapshot holds both.
- Component tests, through the scaffold: `announced` is `["ws", "index"]` for `ws index NAME` and for
  `ws index --bogus`, `["ws"]` for `ws --help` and for `ws nope`, and `None` for `--version`, `--help`, `--bogus`,
  `nope`, `-v -v ws index NAME`, and no arguments.
- `crates/cli/tests/trace.rs`, the built binary in an isolated tree with a `Collector`, the kill switch cleared:
  - `ws index NAME` with a sampled `TRACEPARENT` and a `WINTER_LAUNCH_TIME` written with a comma: exit 0, stderr empty,
    one request; its span is named `winter ws index`, has the trace id and the parent of the `TRACEPARENT`, starts at
    the launch time, and carries only `winter.command`; the resource's `service.name` is `winter`;
  - `ws index` with no name: exit 2; the span's attributes are exactly `winter.command` and `error.type` of `Usage`, and
    its status is code 2 with an empty message;
  - `--version`, `nope`, and no arguments: no connection;
  - the kill switch left at `true`, and a blank endpoint: no connection; an unsampled `TRACEPARENT`: no request;
  - `Answer::Hang`, `Answer::Status(500)`, and a refused endpoint: stdout, stderr, and the exit code are those of the
    same command with tracing off, and the process ends within 2 s;
  - `-v` with a refused endpoint: stderr holds a record of origin `trace`;
  - the collector named with an `https` URL: a connection arrives, no request is recorded, the exit code and stdout are
    unchanged, stderr is empty without `-v`, and the process ends within 2 s;
  - `OTEL_EXPORTER_OTLP_ENDPOINT` and `OTEL_EXPORTER_OTLP_TRACES_ENDPOINT` naming a second collector beside the Winter
    endpoint: the first collector holds the one request and the second sees no connection; the same two variables with
    no Winter endpoint: no connection at either.
- A unit test in `crates/cli/src/trace.rs` that starts a child test of the same test binary through `invoke`: the child
  installs the interrupt handler, announces a command to a `Tracer` whose endpoint is the parent's `Collector`, writes a
  ready line, and raises SIGINT on itself; the child exits 1 and the collector holds no request.
- `grep -rn 'unsafe' crates/cli/src` prints lines of `interrupt.rs` and `trace.rs` only, each block preceded by a
  comment; `grep -rn 'opentelemetry' crates/core crates/test-support crates/dev` prints nothing.
- `git diff --stat` of the change names `crates/cli/Cargo.toml` and `Cargo.lock`, and no other manifest.
- After the change, `cargo fetch --locked` followed by `cargo dev gate full` with no network exits 0.

## Out of scope

- Spans of single operations, `winter.env`, trace context in a child's environment, and the dashboard's spans:
  [tracing.md](../spec/tracing.md), System, names the owner of each.
- A collector whose certificate chains to a private authority: not reachable over `https`, under
  `decision:parallel-workspaces/component-runtime/trace-export-trust-roots`. A completed TLS handshake in a test:
  [tracing.md](../spec/tracing.md), Test seams.
- A bound on name resolution: [tracing.md](../spec/tracing.md), Open questions.
- A span for a deferred interrupt: built with the first command that returns `Termination::Interrupted`.
