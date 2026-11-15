# Tracing

The command span: when one invocation records it, what it carries, how it is exported, and what a trace collector
receives. Which endings open a span is [results-and-exit-codes.md](./results-and-exit-codes.md), Flows; the process
boundary that ends and exports it is [runtime.md](./runtime.md), The process boundary; the variables are listed with
every other process input in [command-surface.md](./command-surface.md), Process inputs. Reference locations are paths
under `tools/winter-cli/src/winter_cli/` of the `winter-workspace` repository at `b57b020`; `tests/` is
`tools/winter-cli/tests/` there.

## System

```text
# engineering:observability-isolation, decision:observability-shutdown
# decision:parallel-workspaces/component-runtime/trace-export-transport
# decision:parallel-workspaces/component-runtime/trace-exporter
crates/core/src/trace.rs       pure: the settings read from the environment snapshot, the launch time, the export
                               headers, the resource, the span's name, the tracer port `run` calls, and the two log
                               records about tracing. It names no tracing crate.
crates/core/src/termination.rs Termination::error_type
crates/cli/src/trace.rs        the binary's tracer: the OpenTelemetry SDK's provider, the OTLP exporter, the open
                               span, its end, and its one export
crates/test-support/src/otlp.rs    the decoder and the loopback collector of the tests; test-harness.md, Interfaces
```

One invocation records at most one span, the command span, and exports it at most once. The span is built, ended, and
sent by the standard OpenTelemetry Rust crates, which only the binary package depends on
([repository-layout.md](./repository-layout.md), The Cargo workspace). The library and the support package depend on
none of them.

```text
outside this contract — each is specified by the epic that owns the command or the port
  spans of single operations inside a command, and the `winter.env` attribute a command adds to its span
      reference: core/internal/otel_command_tracer.py:197-260, core/tracing.py:9-19
  TRACEPARENT and TRACESTATE placed in a child process's environment, and none placed in a managed service launch
      (engineering:observability-isolation); with the port that starts a child process
  the dashboard's session spans and its background export       epic:agent-ready-operation/workspace-dashboard
  WINTER_LAUNCH_TIME left out of every child's environment      with the port that starts a child process
  a child's environment is built from the snapshot, never inherited from the process: the binary removes variables
      from its own process environment (The binary's part), and the snapshot still holds them
                                                                with the port that starts a child process
```

## Domain

### When tracing is on

```text
# reference: cli.py:366-378, core/tracing.py:41-55 — tests/test_cli_tracing.py, tests/core/test_tracing.py
read once, from the environment snapshot, before the command line is parsed
  WINTER_OTEL_EXPORTER_OTLP_ENDPOINT    blank (unset, empty, or white space alone)     -> off
  OTEL_SDK_DISABLED                     `true` after trimming, in any letter case       -> off
                                        any other value, `1` and `yes` included         -> no effect
  otherwise                                                                             -> on, or unavailable
no effect        OTEL_EXPORTER_OTLP_ENDPOINT, OTEL_EXPORTER_OTLP_TRACES_ENDPOINT, OTEL_EXPORTER_OTLP_HEADERS, every
                 other OTEL_EXPORTER_OTLP_*, OTEL_TRACES_SAMPLER, OTEL_TRACES_EXPORTER, TRACESTATE: a generic
                 exporter setting neither switches tracing on nor alters the request. The reference tool hides the
                 OTEL_EXPORTER_OTLP_ and OTEL_PYTHON_EXPORTER_OTLP_ variables from its exporter while it builds it
                 (core/internal/otel_command_tracer.py:43-58); how the binary does it is The binary's part.
off              nothing of the tracing stack is constructed: no provider, no exporter, no HTTP client, no TLS
                 configuration, no thread. No span is recorded, no socket is opened, no request is sent, and no
                 record is written. The crates are linked into the binary and cost it size, not start-up work.
unavailable      tracing is on and the exporter cannot be built as configured: the endpoint value is not UTF-8, or
                 the exporter refuses it. Nothing is recorded or sent, and the invocation is otherwise untouched;
                 one Debug record says why (Records below).
                 reference: container.py:120-133
a variable whose value is not UTF-8
                 the endpoint: unavailable. The headers, OTEL_SERVICE_NAME, OTEL_RESOURCE_ATTRIBUTES, TRACEPARENT,
                 WINTER_LAUNCH_TIME, OTEL_SDK_DISABLED: read as unset.
```

### Endpoint and headers

```text
# reference: core/internal/otel_command_tracer.py:166-170
endpoint     the value trimmed, with every trailing "/" dropped, followed by "/v1/traces"
             (`http://c:4318/` and `http://c:4318` both give http://c:4318/v1/traces)
             `http` and `https` are both served. Nothing else of the value is judged before the exporter is built:
             a value the exporter cannot use is unavailable, and one it accepts and cannot reach is a failed export.
headers      WINTER_OTEL_EXPORTER_OTLP_HEADERS, blank meaning none
             split at ","; each item split at its first "="; name and value percent-decoded and trimmed; the name
             lower-cased; an item with no "=", an empty name, or a name that is no HTTP token is skipped, and the
             others still apply (reference: opentelemetry/util/re.py:52-93, the liberal form)
             a name given twice keeps its first position and its last value
             the parsed pairs are handed to the exporter, which adds them to its request
TLS          an `https` endpoint is verified against the public roots bundled in the binary and against nothing
             else: no environment variable names a certificate or a bundle, and no system certificate store is
             read. A collector whose certificate chains to a private authority is not reachable over `https`: its
             export is a failed export. decision:parallel-workspaces/component-runtime/trace-export-trust-roots
```

### Parent, sampling, and ids

```text
# reference: core/internal/otel_command_tracer.py:97-106,159 — W3C Trace Context, traceparent
TRACEPARENT  handed, when set and not empty, to the SDK's W3C trace-context propagator as the `traceparent` entry
             of a carrier that holds nothing else, which is what the reference tool does with its SDK's propagator
  valid, sampled flag set      the span joins that trace: trace id from the value, parent span id from the value
  valid, sampled flag clear    the caller did not sample: the span is not recorded and nothing is sent
  absent, empty, or invalid    the span is the root of a new trace, recorded and sent
sampler      parent-based over always-on, set explicitly, so no sampler variable is consulted
ids          a new trace id and the span id come from the SDK's random id generator
```

### Start instant

```text
# reference: core/tracing.py:58-78, cli.py:366-372
WINTER_LAUNCH_TIME   trimmed; "," read as "."; then digits, optionally "." and digits, and nothing else
                     epoch seconds; the fraction is cut or padded to nine digits
                     zero, or later than the instant the settings are read: ignored
start instant        the launch time when it is valid; otherwise the instant `run` announces the command
end instant          the instant the boundary ends the span (Flows)
```

### What the span carries

```text
# reference: core/internal/otel_command_tracer.py:85-94,109-113,181-195, cli.py:138-140,211-248
name         "winter " followed by the command path joined by single spaces: `winter ws index`,
             `winter service start` for the alias. The first word is the literal `winter`, whatever the binary is
             called (decision:parallel-workspaces/component-runtime/program-name). The path is the command path of
             runtime.md, The parse and the run: it stops at the first token that is no command name, which is an
             option, a positional, or an unknown name alike.
kind         internal
attributes   winter.command = the name                          always
             error.type     = a name of the vocabulary below    exactly when the exit code is not 0
             and no other: no argument, option value, path, message, or environment value is ever an attribute
status       error, with an empty message, exactly when error.type is present; otherwise unset
events, links    none
resource     service.name = OTEL_SERVICE_NAME when it is set and not empty
                            else the service.name entry of OTEL_RESOURCE_ATTRIBUTES when it is not empty
                            else `winter`
             every other entry of OTEL_RESOURCE_ATTRIBUTES as a string attribute: `key=value` items split at ",",
             each at its first "="; the key trimmed and not percent-decoded; the value trimmed, then
             percent-decoded; an item with no "=" or an empty key skipped; a key given twice keeps its first
             position and its last value
```

```text
error.type vocabulary                  # decision:parallel-workspaces/component-runtime/span-error-type
  the name of the Termination variant the invocation ended with (results-and-exit-codes.md, Domain), as a literal:
    Reported  Usage  BareGroup  Command  StatusCollection  Config  Repo  Space  Env  Scaffold  Aborted
    Interrupted  Provider  ProviderNotStarted  BrokenPipe  Internal
  Success carries none. The vocabulary is closed: a new Termination variant adds its name here and in
  Termination::error_type, and a name is never renamed or reused. The payload of a variant is never part of the
  value: a usage error is `Usage` whatever its fault.
  The reference tool sends its exception class names in this attribute (cli.py:150-162).
```

### What is function and what is free

| Function: equal to the reference tool, or a registered deviation                         | Free: the SDK's and the exporter's own                 |
| ---------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| Whether a request is sent, and that it is one                                            | The instrumentation scope's name and version           |
| Span name; `winter.command`; presence of `error.type`; status code; empty status message | `telemetry.sdk.*` resource attributes                  |
| Trace id and parent span id under a sampled `TRACEPARENT`; a new root otherwise          | The `flags` and `trace_state` fields of the span       |
| Start instant under a valid `WINTER_LAUNCH_TIME`                                         | The `User-Agent` header and any header the client adds |
| `service.name` and the other resource attributes                                         | Wording of the two records                             |
| Request path, method `POST`, protobuf content type, and configured headers               | The response body                                      |
| The 100 ms cap; no effect on stdout, the exit code, or stderr without a logging switch   | HTTP version and connection handling                   |

Registered deviations: the values of `error.type`, no span for a run SIGINT aborts, and the trust roots of an `https`
endpoint ([command-surface.md](./command-surface.md), Registered deviations).

## Interfaces

### The library's part

```rust
// crates/core/src/trace.rs — public through the facade: the binary is the caller. Pure: no clock, no socket, no
// file, no tracing crate; every input is an argument. engineering:library-output-boundary
pub enum TraceSettings {
    Off,
    Unavailable(String),          // why, for the Debug record
    On(TraceConfig),
}
pub struct TraceConfig {
    pub endpoint: String,                         // ends in /v1/traces
    pub headers: Vec<(String, String)>,
    pub resource: Vec<(String, String)>,          // service.name first
    pub traceparent: Option<String>,              // the variable's value, for the propagator; None: a new root
    pub launch_time_unix_nano: Option<u64>,
}
pub fn settings(env: &ProcessEnv, now_unix_nano: u64) -> TraceSettings;
pub fn span_name(command_path: &[String]) -> String;       // "winter ws index"

// The port `run` calls. `&self`: the binary's tracer is shared with the boundary, which ends the span.
pub trait CommandTracer {
    // Called at most once, at step 2 of run_with (runtime.md, The parse and the run): at Stage::Command, after
    // logging is resolved, before the command's `run`, a command's --help, and a command-stage usage error alike.
    fn command_started(&self, command_path: &[String], log: TraceLog);
}
pub struct NoTrace;                               // the tracer of a test that asserts nothing about tracing
impl CommandTracer for NoTrace { /* nothing */ }

// The invocation's resolved logging, as much of it as the boundary needs to report about tracing.
#[derive(Clone, Copy)]
pub struct TraceLog(/* Log */);
impl TraceLog {
    pub fn export_failed(self, err: &mut dyn std::io::Write, cause: &dyn std::fmt::Display);
    pub fn unavailable(self, err: &mut dyn std::io::Write, cause: &dyn std::fmt::Display);
}

// crates/core/src/termination.rs
impl Termination {
    pub fn error_type(&self) -> Option<&'static str>;   // None exactly when exit_code() is 0
}
```

```text
Records                                 # runtime.md, Diagnostics on the stderr handle
  export_failed    Level::Error, origin "trace", only_when_switched: the exporter answered an error — the request
                   was not delivered within the cap, or the answer was no success status
  unavailable      Level::Debug, origin "trace", only_when_switched
  both go through diagnostics::record, so neither is written without -v or a valid WINTER_LOG_LEVEL, and stderr of
  an invocation with no logging switch is the same with tracing on, off, failing, or unavailable
  the crates write nothing themselves: their internal-logging feature is not enabled, and the binary installs no
  subscriber for it
  reference: container.py:120-133; tests/test_cli_otel_process.py (the two "surfaces opentelemetry diagnostics" tests)
```

### The binary's part

```text
crates and features                     # repository-layout.md, The Cargo workspace, holds the manifest rows
  opentelemetry          the API: span, status, attributes, context, the W3C propagator's carrier
  opentelemetry_sdk      the tracer provider, the sampler, the resource, the W3C trace-context propagator, the
                         unbatched span processor, the span exporter trait
  opentelemetry-otlp     the OTLP span exporter over HTTP with protobuf, with a blocking HTTP client and TLS through
                         rustls with bundled public roots
  each at the newest release that builds on the pinned toolchain, 1.99.1, with default features off and only the
  features this contract uses on. The feature names expected are `trace` on all three and, on the exporter,
  `http-proto`, `reqwest-blocking-client`, and the rustls feature with bundled web roots; they are written from the
  crates' documentation as remembered and were not read in a registry, so the change that adds the crates confirms
  each name against the release it takes and records the names it used in the dependency table.
why these choices                       # decision:parallel-workspaces/component-runtime/trace-exporter
  a blocking client    one request at exit needs no event loop. The binary starts no asynchronous runtime; a helper
                       thread the client keeps for itself is the client's own and ends with the process.
  rustls               no system TLS library is linked or looked up at run time, so the binary stays one file. The
                       bundled public roots are the whole trust set, and no feature that reads a system store is
                       enabled (decision:parallel-workspaces/component-runtime/trace-export-trust-roots).
                       If the TLS stack's cryptography needs a C compiler to build, that is a build prerequisite
                       the change states in context/tooling/development.md.
  no batching          one span per process: the unbatched processor exports on the thread that ends the span
  no retry, no compression, no gRPC: none of those features is enabled
```

```rust
// crates/cli/src/trace.rs
// E is the SDK's span exporter trait: the OTLP exporter in `main`, an exporter of the test's own in a unit test.
pub(crate) struct Tracer<E> { /* the settings; the exporter's builder; in a RefCell: the provider, the open span,
                                 its TraceLog, and the result of the export */ }
impl Tracer<opentelemetry_otlp::SpanExporter> {
    pub(crate) fn new(settings: TraceSettings) -> Self;        // constructs nothing of the stack
}
impl<E: opentelemetry_sdk::trace::SpanExporter + 'static> Tracer<E> {
    // The seam of the unit tests: `build` stands in for the construction of the OTLP exporter.
    pub(crate) fn with_exporter(
        settings: TraceSettings,
        build: Box<dyn FnOnce(&TraceConfig) -> Result<E, String>>,
    ) -> Self;
    // Ends the span when one is open, which exports it, and writes at most one record. Does nothing otherwise.
    pub(crate) fn finish(&self, error_type: Option<&'static str>, stderr: &mut dyn std::io::Write);
}
impl<E: opentelemetry_sdk::trace::SpanExporter + 'static> CommandTracer for Tracer<E> { /* opens the span */ }

// Called by `main` once, directly after the snapshot is taken and only when the settings are On.
pub(crate) fn hide_generic_exporter_variables(env: &ProcessEnv);
```

```text
construction                            # nothing before a command is announced
  Tracer::new keeps the settings and builds nothing. At command_started with settings On, and never earlier:
    the exporter      built for TraceConfig.endpoint with TraceConfig.headers and a timeout of 100 ms
    the provider      the explicit sampler, a resource of exactly TraceConfig.resource and whatever the SDK adds
                      about itself, and the unbatched processor over a wrapper of the exporter that keeps the
                      result of its one export
    the span          named span_name(path), kind internal, attribute winter.command, the parent the propagator
                      reads from TraceConfig.traceparent, started at the launch time or now
  a failure to build the exporter turns the tracer unavailable with the failure's text
  an invocation that ends at the root never announces a command, so it constructs nothing even with tracing on
hide_generic_exporter_variables         # reference: core/internal/otel_command_tracer.py:43-58
  the exporter crate reads OTEL_EXPORTER_OTLP_* from the process environment when it is built and lets them
  override what the program configured. `main` therefore removes from its own process environment every variable
  of the snapshot whose name starts with OTEL_EXPORTER_OTLP_, before any thread exists and before the tracer is
  built. This is the one `unsafe` block of trace.rs (std::env::remove_var), sound because the process is
  single-threaded there. The snapshot is untouched and is what a child's environment is built from (System).
  With tracing off nothing is removed.
finish
  no span open                      nothing, except: unavailable and a command announced -> log.unavailable
  span open                         when error_type is Some: the attribute error.type and an error status with an
                                    empty message. The span is ended at now. For a recorded span the processor
                                    calls the exporter once, on this thread, and `finish` reads the kept result:
                                    an error -> log.export_failed. The provider is then shut down.
  a span the sampler did not record  ending it exports nothing; no result is kept and nothing is written
the cap                                 # decision:observability-shutdown; core/internal/otel_command_tracer.py:35
  the exporter's timeout is the one bound on the export: 100 ms from the start of the request, the value the
  reference tool gives its exporter as "the exit cap: one request, never longer than this". One request, no retry.
  The executable tests hold the bound from outside: a collector that accepts and never answers does not keep the
  process alive (Test seams).
```

### Test seams

```text
unit, crates/core/src/trace.rs       `settings` over literal environments
unit, crates/cli/src/trace.rs        `Tracer::with_exporter` and an exporter of the test's own that records the
                                     span data it is handed, or answers an error; no socket
                                     the real exporter against the loopback collector, in the test process
component                            the scaffold passes a recording CommandTracer and reports what `run` announced
                                     (runtime.md, Test seams)
executable, crates/cli/tests/        the built binary with WINTER_OTEL_EXPORTER_OTLP_ENDPOINT naming the loopback
                                     collector of the test; OTEL_SDK_DISABLED of the pinned environment is cleared
                                     by the test that wants tracing on
the collector                        test-harness.md, Interfaces: a listener on 127.0.0.1 that reads what the
                                     standard exporter sends and decodes the protobuf body with a decoder written in
                                     the support package from the OTLP field table there. The support package takes
                                     no tracing crate: the decoder is an independent reading of the wire format, so
                                     a test compares what the crates sent with what the contract says, not the
                                     crates with themselves.
TLS                                  no automated test completes a TLS handshake: a test would need a certificate
                                     the bundled roots trust. An executable test names the plain collector with an
                                     `https` URL and asserts that a connection arrives and the export fails within
                                     the cap, which shows the scheme is served and bounded.
no scenario                          the pinned environment of a scenario turns tracing off, and a collector is no
                                     dimension a scenario compares
no network                           every test addresses 127.0.0.1 alone; none resolves a name or reaches a real
                                     collector
```

## Flows

```text
# reference: cli.py:211-228,451-455
main      settings(env, now) -> when On: hide_generic_exporter_variables -> Tracer::new        before `run`
run       Stage::Command, logging resolved -> tracer.command_started(path, log)
            on: the exporter, the provider, and the span are built; the start instant is the launch time or now
            an invocation that ends at the root never calls it: no span, no request, no record
conclude  runtime.md, The process boundary, in its order:
            the ending is final once the last flush is classified
            -> the diagnostic and the backtrace are written
            -> tracer.finish(termination.error_type(), stderr)
                 span open         the span ends and is exported; on failure, log.export_failed
                 unavailable       log.unavailable, when a command was announced
                 otherwise         nothing
            -> the exit code, unchanged by anything `finish` did
panic     the span is ended with `Internal` and exported like any other ending
SIGINT    undeferred: the handler exits the process; `finish` never runs and nothing is sent
          deferred: the command returns Termination::Interrupted and the span is exported with `Interrupted`
          decision:parallel-workspaces/component-runtime/interrupt-trace-export
```

Export never changes the result: `finish` returns nothing, runs after the last byte of stdout and the diagnostic are
written, and cannot panic past the boundary; a collector that refuses, hangs, or answers an error costs at most the cap
and leaves stdout, the exit code, and an unswitched stderr as they were (reference:
`core/internal/otel_command_tracer.py:35,383-409`; `tests/test_cli_otel_process.py`, the hanging and refused endpoint
tests).

## Open questions

- Whether the exporter's timeout covers the resolution of a host name is a property of the HTTP client that was not read
  in its source; `decision:observability-shutdown` leaves resolution outside the cap. Engineering answers it when a
  collector is first addressed by name in a deployment.
- The names of the crate features, the way the exporter is given its timeout and headers, that it reads
  `OTEL_EXPORTER_OTLP_*` when built, that the unbatched processor exports on the ending thread, and that the TLS stack
  with bundled roots reads no variable and no system store are stated from the crates' documentation as remembered. The
  change that adds the crates confirms each against the release it takes; a point that does not hold and changes
  anything under Function above goes back to engineering before the change merges.
