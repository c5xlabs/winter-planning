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
# decision:parallel-workspaces/component-runtime/trace-export-total-bound
crates/core/src/trace.rs       pure: the settings read from the environment snapshot, the launch time, the export
                               headers, the resource, the span's name, the tracer port `run` calls, and the three log
                               records about tracing. It names no tracing crate.
crates/core/src/termination.rs Termination::error_type
crates/cli/src/trace.rs        the binary's tracer: the OpenTelemetry SDK's provider, the OTLP exporter, the open
                               span, its end, its one export, and the thread and the wait that bound both
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
                 OTEL_TRACES_SAMPLER_ARG and the SDK's span limits, OTEL_SPAN_ATTRIBUTE_COUNT_LIMIT,
                 OTEL_SPAN_EVENT_COUNT_LIMIT, and OTEL_SPAN_LINK_COUNT_LIMIT, likewise: the binary sets the sampler,
                 the limits, and the resource itself (The binary's part, construction).
on               the stack is built when a command is announced, and never earlier. At the end of the invocation
                 the span is ended and exported on a thread the tracer starts for that, and the ending waits for
                 that thread for the cap at most (The binary's part: finish, the export thread, the cap).
off              nothing of the tracing stack is constructed: no provider, no exporter, no HTTP client, no TLS
                 configuration, no thread. No span is recorded, no socket is opened, no request is sent, and no
                 record is written. The crates are linked into the binary and cost it size, not start-up work.
unavailable      tracing is on and the exporter cannot be built: the endpoint value is not UTF-8, the exporter
                 refuses it, or the trust store yields no certificate, whatever the endpoint's scheme (TLS below).
                 Nothing is recorded or sent, and the invocation is otherwise untouched: stdout and the exit code
                 are those of the same command with tracing off, and one Debug record says why (Records below).
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
             two kinds of parsed pair are left out of the export, and the others still apply. Both are judged on
             the pairs as parsed, after a name given twice has taken its last value
             (decision:parallel-workspaces/component-runtime/trace-header-precedence)
               the name content-type          ignored: the export always carries the protobuf content type.
                                              No record is written for it.
               a value that is not legal      skipped, and named in a Debug record by its name alone, never by its
               in an HTTP header              value (Records below). Not legal is a value that holds a byte below
                                              0x20 other than the tab, or the byte 0x7F: the rule by which the
                                              exporter's HTTP crate refuses a value, as read in release 1.5.0 of
                                              `http` (header/value.rs). A byte of 0x80 and above is legal.
             the pairs that remain are handed to the exporter, which adds them to its request; each value is sent
             as parsed here, decoded once and never a second time (The binary's part, construction)
             a configured user-agent is such a pair and replaces the exporter's own
             (decision:parallel-workspaces/component-runtime/trace-user-agent)
             three inputs are read differently from the reference's helper, a registered deviation
             (decision:parallel-workspaces/component-runtime/trace-header-parsing)
               input                          the reference                    this tool
               authorization=Bearer abc%3D    the value sent undecoded         the value sent decoded
               x-a=a;b                        the item skipped                 the item sent
               bad%3Aname=2                   sent under the name bad:name     the item skipped
TLS          an `https` endpoint is verified against the operating system's trust store, as the platform's
             certificate verifier reads it; the binary bundles no roots. A collector whose certificate chains to a
             private authority installed in that store is reachable over `https`; one whose authority is not there
             is a failed export. No variable of this tool names a certificate or a bundle.
             the store is read when the exporter is built, at the announcement of the command, and for an `http`
             endpoint as for an `https` one: the exporter's HTTP client builds its TLS configuration whatever the
             scheme. A store that yields no certificate — none installed, as in a container image without CA
             certificates, or an empty file named by a variable below — keeps the exporter from being built, so
             tracing is unavailable on that machine for both schemes (When tracing is on).
             SSL_CERT_FILE, SSL_CERT_DIR: read by the certificate loader under the verifier, as read for Linux.
             When either is set, the certificates are those of the file and of the directory named, and the
             system's own locations are not read. Neither is a variable of this tool; both are process inputs
             (command-surface.md, Process inputs).
             the reference tool's HTTP library is believed to verify against its runtime's default trust locations,
             the system certificate store among them: the two agree wherever those locations are the system's
             store, and a difference between them is a registered deviation
             decision:parallel-workspaces/component-runtime/trace-export-trust-roots
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
start instant        the launch time when it is valid, which the binary hands to the SDK. Otherwise no source of
                     the binary reads one: the SDK reads the clock when it builds the span, at the announcement of
                     the command, and only for a span the sampler records (read in opentelemetry_sdk 0.33.1,
                     trace/tracer.rs). For a span the sampler does not record no instant is read.
end instant          the instant the boundary ends the span (Flows): `finish` reads it before it starts the export
                     thread, so the span's duration holds nothing of the export
the clock            the sources of the binary read it in two files: main.rs, for the instant the settings are read
                     at, and trace.rs, for the end instant. The library reads none.
                     decision:parallel-workspaces/component-runtime/clock-readers
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
| Whether a request is sent, and that it is one unless the collector answers a redirect    | The instrumentation scope's name and version           |
| Span name; `winter.command`; presence of `error.type`; status code; empty status message | `telemetry.sdk.*` resource attributes                  |
| Trace id and parent span id under a sampled `TRACEPARENT`; a new root otherwise          | The `flags` and `trace_state` fields of the span       |
| Start instant under a valid `WINTER_LAUNCH_TIME`                                         | The `User-Agent` header and any header the client adds |
| `service.name` and the other resource attributes                                         | Wording of the three records                           |
| Request path, method `POST`, protobuf content type, and configured headers               | The response body                                      |
| The 100 ms cap; no effect on stdout, the exit code, or stderr without a logging switch   | HTTP version and connection handling                   |

Registered deviations: the values of `error.type`, no span for a run SIGINT aborts, the trust roots of an `https`
endpoint, no tracing on a machine whose trust store yields no certificate, and the three inputs of the configured
headers that Endpoint and headers lists ([command-surface.md](./command-surface.md), Registered deviations).

Following a redirect and the two rules for a configured header are intended behaviour, under
`decision:parallel-workspaces/component-runtime/trace-export-redirects` and
`decision:parallel-workspaces/component-runtime/trace-header-precedence`. What the reference tool does in either case
was read in no source: its HTTP library is believed to follow a redirect as well. Neither is registered as a deviation.

An export that goes through the proxy a proxy variable names is as built, under
`decision:parallel-workspaces/component-runtime/trace-export-proxies` (The binary's part, proxies). The reference tool's
HTTP library is believed to honour the same variables, which was read in no source, and no deviation is registered.

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
    pub headers: Vec<(String, String)>,           // what is sent: no content-type, no value illegal in a header
    pub skipped_headers: Vec<String>,             // the names left out for an illegal value, in configured order
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
    pub fn header_skipped(self, err: &mut dyn std::io::Write, name: &str);
}

// crates/core/src/termination.rs
impl Termination {
    pub fn error_type(&self) -> Option<&'static str>;   // None exactly when exit_code() is 0
}
```

```text
Records                                 # runtime.md, Diagnostics on the stderr handle
  export_failed    Level::Error, origin "trace", only_when_switched: the span was not exported within the cap. The
                   exporter answered an error — the request was not delivered within its timeout, or the answer
                   was no success status — or the export thread did not complete in time, panicked, or could not
                   be started (The binary's part, finish)
  unavailable      Level::Debug, origin "trace", only_when_switched
  header_skipped   Level::Debug, origin "trace", only_when_switched: a configured header was left out of the export
                   because its value is not legal in an HTTP header (Endpoint and headers). The record holds the
                   name of the header and never its value.
                   decision:parallel-workspaces/component-runtime/trace-header-precedence
  the levels: decision:parallel-workspaces/component-runtime/trace-record-levels
  all three go through diagnostics::record, so none is written without -v or a valid WINTER_LOG_LEVEL, and stderr
  of an invocation with no logging switch is the same with tracing on, off, failing, or unavailable
  the crates write nothing themselves: their internal-logging feature is not enabled, and the binary installs no
  subscriber for it
  reference: container.py:120-133; tests/test_cli_otel_process.py (the two "surfaces opentelemetry diagnostics" tests)
```

### The binary's part

```text
crates and features                     # repository-layout.md, The Cargo workspace, holds the manifest lines
  opentelemetry          the API: span, status, attributes, context, the W3C propagator's carrier
  opentelemetry_sdk      the tracer provider, the sampler, the resource, the W3C trace-context propagator, the
                         unbatched span processor, the span exporter trait
  opentelemetry-otlp     the OTLP span exporter over HTTP with protobuf, with a blocking HTTP client and TLS through
                         rustls, verified by the platform's certificate verifier
  each at 0.33.1, the newest release, with default features off
    opentelemetry        features: trace
    opentelemetry_sdk    features: trace
    opentelemetry-otlp   features: trace, http-proto, reqwest-blocking-client, reqwest-rustls
  read in release 0.33.1 of the exporter:
    http-proto               OTLP over HTTP with protobuf. It switches the exporter's `metrics` feature on, and with
                             it `metrics` of the API and of the SDK: more than this contract uses, and not
                             avoidable. Nothing of metrics is constructed.
    reqwest-blocking-client  the blocking client of reqwest 0.13.5. reqwest depends on tokio unconditionally, and
                             its blocking client runs a runtime on a helper thread of its own. Dropping the last
                             handle of the client joins that thread. The exporter builds the client with a timeout
                             and nothing else, so it follows redirects, ten at most, which is the client's default.
                             What the timeout covers is under the cap, and what a redirect does under redirects,
                             below.
    reqwest-rustls           reqwest's `rustls` feature: rustls over aws-lc-rs, verifying through
                             rustls-platform-verifier, which reads the platform's trust store. It is the exporter's
                             one rustls feature for HTTP: it has none with bundled web roots, its `tls-*` features
                             serve gRPC alone, and with no TLS feature `https` is not served at all.
  the feature `trace` exists on the three crates at 0.33.1. That each builds with default features off is not
  shown: the change that adds the crates confirms it (Open questions).
why these choices                       # decision:parallel-workspaces/component-runtime/trace-exporter
  a blocking client    one request at exit needs no event loop in the binary. The binary and the library start no
                       asynchronous runtime and name none in their sources. The client runs one of its own on a
                       helper thread it keeps for itself, which ends with the process, so `tokio` is in Cargo.lock.
  rustls               no system TLS library is linked or looked up at run time, so the binary stays one file. The
                       trust roots are not in the binary: the platform's verifier reads the operating system's
                       trust store at run time, once for each invocation that builds the exporter, when the client
                       is constructed and whatever the endpoint's scheme. The cryptography under rustls,
                       aws-lc-sys, compiles bundled C in its build script, so a C compiler is a build
                       prerequisite, which the change states in context/tooling/development.md.
                       decision:parallel-workspaces/component-runtime/trace-export-trust-roots
  no batching          one span per process: the unbatched processor exports on the thread that ends the span,
                       which is the export thread of `finish`
  no retry             the exporter retries by default in this release, three times and first after 100 ms. The
                       binary hands it the disabled retry policy, so that the one request is one by construction
                       and not because the cap leaves a second no room.
  no compression, no gRPC: neither feature is enabled
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
    // Ends the span when one is open — on the export thread, which exports it and is waited for no longer than
    // the cap — and writes at most one record about it, after one for each header skipped. Does nothing otherwise.
    pub(crate) fn finish(&self, error_type: Option<&'static str>, stderr: &mut dyn std::io::Write);
}
impl<E: opentelemetry_sdk::trace::SpanExporter + 'static> CommandTracer for Tracer<E> { /* opens the span */ }

// Called by `main` once, directly after the snapshot is taken and only when the settings are On.
pub(crate) fn hide_generic_exporter_variables(env: &ProcessEnv);
```

```text
construction                            # nothing before a command is announced
  Tracer::new keeps the settings and builds nothing. At command_started with settings On, and never earlier:
    the exporter      built for TraceConfig.endpoint, with the protocol HTTP with protobuf named explicitly,
                      TraceConfig.headers, a timeout of 100 ms, which is the inner bound of the cap below, and the
                      disabled retry policy
                      the exporter percent-decodes every header value it is handed, and `settings` has decoded
                      each already. The binary therefore hands every value with each "%" written "%25", and the
                      name as it is, so that the value sent is the value of TraceConfig byte for byte. The escape
                      was taken over an HTTP client of the binary's own that carries the headers, which would
                      make that client a direct dependency of the binary.
                      one name is respelled: `user-agent` is handed as `User-Agent`, the key the exporter keeps its
                      own under, so that which of the two values is sent does not hang on the order of a map
                      (decision:parallel-workspaces/component-runtime/trace-user-agent)
                      left to itself the exporter lets a configured content-type replace the protobuf one, which
                      it sets first, and drops a header whose value is not legal without a word. TraceConfig.headers
                      holds neither kind (Endpoint and headers), so the exporter is handed nothing it replaces its
                      content type with and nothing it drops.
    the provider      the explicit sampler; span limits given explicitly, 128 attributes, 128 events, and 128
                      links for a span, which are the SDK's defaults written out; a resource built from the empty
                      one, with no detector, that holds exactly TraceConfig.resource; and the unbatched processor
                      over a wrapper of the exporter that keeps the result of its one export
                      left to its defaults, the SDK would read OTEL_TRACES_SAMPLER, OTEL_TRACES_SAMPLER_ARG,
                      OTEL_SPAN_ATTRIBUTE_COUNT_LIMIT, OTEL_SPAN_EVENT_COUNT_LIMIT, OTEL_SPAN_LINK_COUNT_LIMIT,
                      OTEL_SERVICE_NAME, and OTEL_RESOURCE_ATTRIBUTES. With the three set here none of them
                      alters what is sent: the last two reach the resource through `settings` alone, and an
                      attribute limit of 0 or 1 drops neither winter.command nor error.type.
    the span          named span_name(path), kind internal, attribute winter.command, the parent the propagator
                      reads from TraceConfig.traceparent, started at the launch time or now
  a failure to build the exporter turns the tracer unavailable with the failure's text; a trust store that yields
  no certificate is such a failure
  an invocation that ends at the root never announces a command, so it constructs nothing even with tracing on
hide_generic_exporter_variables         # reference: core/internal/otel_command_tracer.py:43-58
  the exporter crate reads OTEL_EXPORTER_OTLP_* from the process environment when it is built: it lets the
  endpoint, timeout, compression, and protocol variables stand in where the program set none, and adds the header
  variables over the programmed headers. `main` therefore removes from its own process environment every variable
  of the snapshot whose name starts with OTEL_EXPORTER_OTLP_, before any thread exists and before the tracer is
  built. This is the one `unsafe` block of trace.rs (std::env::remove_var), sound because the process is
  single-threaded there. The snapshot is untouched and is what a child's environment is built from (System).
  With tracing off nothing is removed.
finish
  no span open                      nothing, except: unavailable and a command announced -> log.unavailable
  span open                         on the ending thread: when error_type is Some, the attribute error.type and an
                                    error status with an empty message; then the end instant is read. The span,
                                    the provider, and that instant go to the export thread, which `finish` starts
                                    and waits for (the export thread, below). What the wait answers decides the
                                    one record, and `finish` writes it on the ending thread:
                                      exported, or nothing to export     nothing
                                      the exporter answered an error     log.export_failed, the exporter's text
                                      the thread panicked                log.export_failed, the panic's message
                                      the cap expired                    log.export_failed, "the export exceeded
                                                                         its time of 100 ms"
                                      the thread could not be started    log.export_failed, the system's error
  a span the sampler did not record  goes the same way: ending it exports nothing, the thread answers that there was
                                    nothing to export, and nothing is written
  skipped headers                   with a span open, recorded or not, `finish` writes one log.header_skipped for
                                    each name of TraceConfig.skipped_headers, in their order, on the ending thread
                                    and before the record of the export. With no span open — tracing off or
                                    unavailable, the exporter not built, or no command announced — it writes none:
                                    nothing was to be sent.
the export thread                       # decision:parallel-workspaces/component-runtime/trace-export-total-bound
  the one thread the sources of the binary start, named `trace-export`, started by `finish` and only for an open
  span: with tracing off, unavailable, or no command announced, none is started
  what it owns      the span and the provider, outright. `finish` and `main` keep no handle of the provider, of
                    the exporter, or of the HTTP client: each of those ends by being dropped on the export thread,
                    so no drop on the ending thread or in `main` can join the client's helper thread.
  what it runs      in this order, inside one unwind catch:
                      1. ends the span at the instant it was handed; for a recorded span the processor calls the
                         exporter once, on this thread
                      2. reads the result the wrapper kept
                      3. shuts the provider down; what a shutdown answers changes nothing
                      4. drops the provider and with it the exporter and the client, which joins the helper thread
                      5. sends its answer on a channel — exported or nothing to export, the exporter's error, or
                         the panic — as the last thing it does
  a panic there     never reaches stderr and never changes the ending. The thread arms the panic slot of its own
                    id before step 1 and empties it before step 5 (runtime.md, The process boundary, panic hook),
                    so the hook writes nothing for it, under the hook of `main` and under that of a unit test
                    alike; the unwind is caught, and the answer is a failed export whose cause is the stored
                    message. The rule of runtime.md that the thread joining a panicked thread turns the join error
                    into InternalError::Panic does not apply: nothing joins this thread for its panic, and a
                    panic of the export is a failed export, never an internal failure of the command. A thread
                    abandoned at the cap stays armed, so a panic after the expiry writes nothing either.
  the wait          a receive on that channel with a timeout of the cap, begun once the thread is started
                      an answer arrives     `finish` joins the thread, which has nothing left to do, and reports
                      the cap expires       `finish` abandons the thread: it drops the handle and does not join.
                                            The thread runs on with everything it owns and ends with the process.
                                            Whatever it answers later is read by no one.
                      the channel closes with no answer   reported as a failed export; the thread is joined
  a failed start    the span and the provider are not dropped on the ending thread, where dropping the span would
                    end it and export it with no outer bound: they are handed through a slot the thread takes
                    them from, and after a failed start `finish` forgets what the slot still holds
the cap                                 # decision:observability-shutdown; core/internal/otel_command_tracer.py:35
  100 ms, the value the reference tool gives its exporter as "the exit cap: one request, never longer than this".
  One request: retry is disabled. The cap is held twice, and the outer bound is the one this contract promises.
  outer, the binary's     the wait of `finish`: 100 ms from the start of that wait, whatever the export thread is
                          doing by then — resolving the host name, connecting, the TLS handshake, sending the
                          request, receiving the response's head or its body, following a redirect, shutting the
                          provider down, or joining the client's helper thread. `finish` costs the invocation that
                          wait and the start of one thread, and nothing a collector or a resolver does adds to it.
  inner, the exporter's   the exporter's timeout, 100 ms, which reaches the blocking client's `timeout` and nothing
                          else. Read in reqwest 0.13.5 and opentelemetry-http 0.33.1, it is no bound on the export:
                            blocking/client.rs     one wait covers connecting, TLS, sending the body, and receiving
                                                   the response's head
                            blocking/response.rs   every read of the response's body then has a wait of its own, of
                                                   the same length, started afresh
                            opentelemetry-http     the body is read to its end, up to 4 MiB and one byte
                            blocking/client.rs     a lookup that hangs is abandoned by the wait, and the drop of the
                                                   client then joins the helper thread, which waits for the lookup
                          so under it alone a collector that answers its head and stalls costs two caps, one that
                          sends a byte of its body every 90 ms holds the exit for hours, and a resolver that hangs
                          holds it for the resolver's own timeout. The exporter's builder has no switch for any of
                          the three. The inner bound stays at the cap: it ends an ordinary slow export with the
                          exporter's own cause, and it ends the export thread's work where it can.
  which comes first       for a collector that never answers both run out together, and either may be reported: the
                          record is the same record with one cause or the other, and no test asserts which
  on expiry               the span is as it was ended: an expiry adds no attribute, no status, and no second span.
                          The request may be out in full, in part, or not at all, so a collector may still receive
                          the span the record calls not exported. stdout is complete and flushed before `finish`
                          runs, and the exit code is the ending's: an expiry changes neither.
redirects                               # decision:parallel-workspaces/component-runtime/trace-export-redirects
  followed, as the client follows them by default: the exporter builds the client with a timeout and nothing else,
  and its builder has no switch for the redirect policy. Read in reqwest 0.13.5 (redirect.rs, async_impl/client.rs,
  blocking/client.rs) and in tower-http 0.6.11 (follow_redirect), through which that client follows a redirect:
  how many          ten in a row are followed; an eleventh ends the request with an error, which is a failed export
  301, 302          the POST is sent on as a GET with no body, and without the headers content-type,
                    content-length, content-encoding, and transfer-encoding
  303               the same: a GET with no body, without those four headers
  307, 308          the method and the body stay: the POST is sent on with the span
  not followed      an answer of another status, and a redirect whose Location is absent or cannot be read as an
                    address, is the answer of the export. A 3xx left so is no success status: a failed export.
  the next address  `http` or `https`; any other scheme ends the request with an error
  headers           every other header of the request goes on to the next address, the configured ones among them,
                    with two changes. When the next address differs from the one before it in host, port, or
                    scheme, the headers authorization, cookie, cookie2, proxy-authorization, and www-authenticate
                    are left off. And a Referer header is added that names the address before, without user,
                    password, or fragment, unless the step is from `https` to `http`.
  what arrives      the export is judged by the last answer alone. After a 307 or a 308 the span arrives at the
                    address the collector named. After a 301, a 302, or a 303 no span arrives anywhere: what is
                    sent on is a GET without it, and a success status in answer to that GET counts as exported, so
                    no record is written.
  the bound         the outer bound of the cap covers every step
  the reference     what its HTTP library does with a redirect was read in no source; it is believed to follow one
                    as well
  read and not run: no test has a collector answer a redirect (Open questions)
proxies                                 # decision:parallel-workspaces/component-runtime/trace-export-proxies
  honoured, as the client honours them by default: the exporter builds the client with a timeout and nothing else,
  so the client takes its proxies from the environment. Read in reqwest 0.13.5 (proxy.rs, async_impl/client.rs)
  and in hyper-util 0.1.21 (client/proxy/matcher.rs), which reads the variables for that client:
  the variables     HTTP_PROXY, HTTPS_PROXY, ALL_PROXY, and NO_PROXY, each looked up in capitals first and then in
                    lower case, when the client is built, at the announcement of the command. With REQUEST_METHOD
                    set, HTTP_PROXY is ignored (command-surface.md, Process inputs).
  what it does      when a variable names a proxy for the endpoint, the request goes to the proxy and not to the
                    endpoint's host. A user name and a password in the proxy's address are sent to the proxy, in
                    a proxy-authorization header of the Basic scheme.
  observed          with HTTP_PROXY=http://user:password@127.0.0.1:<port> naming a second loopback collector, and
                    an `http` endpoint: the endpoint's collector received nothing, and the second received
                    `POST http://127.0.0.1:<port of the endpoint>/v1/traces` with `proxy-authorization: Basic …`
  read and not run  HTTPS_PROXY, ALL_PROXY, NO_PROXY, the lower-case names, and an `https` endpoint through a proxy
  the bound         the outer bound of the cap covers the way through a proxy like every other step
  the reference     its HTTP library is believed to honour the same variables; that was read in no source
  no test names a proxy (Open questions)
at exit
  `main` returns std::process::ExitCode (runtime.md, The process boundary); no source of the binary calls
  std::process::exit. On that return the Rust runtime ends the process through the C library's `exit`, which waits
  for no thread: a thread nobody joined ends with the process, at once. After an expiry the export thread is alive
  at that point, and so may be the client's helper thread it is waiting for; both end there. After an export that
  completed, the export thread was joined and had joined the helper thread, so no thread of tracing is left.
  The binary therefore does not promise that no thread outlives `main`. It promises that `main` waits for none
  after the cap, and nothing may be added that does: no scoped thread around the export, no handle of the provider,
  the exporter, or the client held where its drop runs on the ending thread or in `main`, no join of the export
  thread after an expiry, and no exit handler that waits.
  an interrupt is unchanged: the handler ends the process with `_exit` on whichever thread the signal lands,
  during the wait of `finish` as anywhere else (runtime.md, Interrupts and signals)
```

### What a user is told

```text
# decision:parallel-workspaces/component-runtime/tracing-user-notice
README.md      a section for users (repository-layout.md, Root after the wipe) that states, each as this spec has it:
  tracing exists      a command can send one trace span to a collector, and does so only when switched on
  the switch          WINTER_OTEL_EXPORTER_OTLP_ENDPOINT, set and not blank. The installed `winter` reads the same
                      variable, so a shell set up for that tool switches tracing on for this binary as well.
  the kill switch     OTEL_SDK_DISABLED=true
  ignored             the generic OTEL_EXPORTER_OTLP_* variables neither switch tracing on nor alter the request
  where               one POST to the endpoint followed by /v1/traces, at the end of each invocation that names a
                      command or a group, its --help and its usage errors included. --version, root help, and a
                      command line that names neither send nothing, and nothing is sent under a TRACEPARENT whose
                      sampled flag is clear or when the exporter cannot be built.
  what is sent        the span's name, `winter` and the command path, and the same text as winter.command;
                      error.type, one name of the closed vocabulary, when the exit code is not 0; the start and
                      end instants of the span, the start being the value of WINTER_LAUNCH_TIME when that is
                      valid, and its ids, the trace id and the parent of a sampled TRACEPARENT among them;
                      service.name and every entry of OTEL_RESOURCE_ATTRIBUTES; the headers of
                      WINTER_OTEL_EXPORTER_OTLP_HEADERS, which may hold credentials, with the two exceptions
                      below; and what the SDK and the HTTP client add of their own (What is function and what is
                      free)
  headers not sent    a configured content-type, since the export always carries the protobuf one; and a header
                      whose value is not legal in an HTTP header, which is named, without its value, in a log
                      record at the debug level (Endpoint and headers)
  what is never sent  an argument, an option value, a path, a message, an environment value other than those this
                      list names, or anything a command wrote
  redirects           a redirect the collector answers is followed, so the request and the configured headers can
                      go to an address the collector names (The binary's part, redirects)
  proxies             the proxy variables HTTP_PROXY, HTTPS_PROXY, ALL_PROXY, and NO_PROXY, in capitals or in lower
                      case, are honoured: the export goes through the proxy they name for the endpoint, and a
                      user name and a password in that proxy's address are sent to the proxy (The binary's part,
                      proxies)
  the bound           the ending of a command waits for the export for 100 ms at most
  a failed export     changes neither stdout nor the exit code, and is reported only as a log record under a
                      logging switch: -v, or a valid WINTER_LOG_LEVEL at ERROR or below
               the same section gives, each from its owner:
  exit codes          0, 1, and 2 as results-and-exit-codes.md, Exit codes, has them for the commands that exist,
                      and that an interrupt ends with a notice on stderr and exit 1, not 130 (Interrupts there),
                      except in a process started with the interrupt signal ignored, which leaves it ignored, so
                      that an interrupt does nothing to it (runtime.md, Interrupts and signals;
                      decision:parallel-workspaces/component-runtime/interrupt-ignored-at-start)
  WINTER_LOG_LEVEL    the names it accepts, the threshold without it, and that any other value is ignored
                      silently (runtime.md, Logging)
root help      one line, after the options, that names WINTER_OTEL_EXPORTER_OTLP_ENDPOINT as the variable that makes
               a command send a trace span and README.md as the place that says what is sent. Its wording is free
               (command-surface.md). The help of a group and of a command carries no such line. The diagnostic of
               a bare root is the root help and carries it too.
```

### Test seams

```text
unit, crates/core/src/trace.rs       `settings` over literal environments
unit, crates/cli/src/trace.rs        `Tracer::with_exporter` and an exporter of the test's own that records the
                                     span data it is handed, answers an error, panics, or does not return before
                                     the test releases it; no socket
component                            the scaffold passes a recording CommandTracer and reports what `run` announced
                                     (runtime.md, Test seams)
executable, crates/cli/tests/        the built binary with WINTER_OTEL_EXPORTER_OTLP_ENDPOINT naming the loopback
                                     collector of the test; OTEL_SDK_DISABLED of the pinned environment is cleared
                                     by the test that wants tracing on
the real exporter                    is built only in a child a test starts through `invoke` under the pinned
                                     environment, and never in a test process. That child is the built binary of an
                                     executable test, which is where a claim about the real exporter is tested by
                                     preference, or a child test of the binary package's own test binary where the
                                     subject ends its own process (test-harness.md, Tiers that use it). The
                                     child's environment is cleared and then explicit, so no proxy variable, no
                                     OTEL_* variable, and no SSL_CERT_* variable of the developer reaches the
                                     exporter; built in the test process it would inherit all of them.
the trust store of a test            a test that clears the kill switch sets SSL_CERT_FILE, through PinnedEnv::set,
                                     to a file it wrote under <ROOT>/tmp. The file holds the text of
                                     crates/cli/tests/data/trust-root.pem, compiled into the test: one self-signed
                                     certificate made for this purpose, whose key was discarded. The exporter then
                                     reads that file and nothing of the host's store, so the test touches nothing
                                     outside its tree, needs no exception to that rule, and passes on a machine
                                     that has no store. No test completes a handshake, so the certificate judges
                                     nothing. The test of a machine without a store names an empty file of its tree
                                     instead and expects tracing unavailable. No test sets SSL_CERT_DIR.
the collector                        test-harness.md, Interfaces: a listener on 127.0.0.1 that reads what the
                                     standard exporter sends and decodes the protobuf body with a decoder written in
                                     the support package from the OTLP field table there. The support package takes
                                     no tracing crate: the decoder is an independent reading of the wire format, so
                                     a test compares what the crates sent with what the contract says, not the
                                     crates with themselves.
the cap, unit                        an exporter that does not return: `finish` returns, with the export record
                                     under a switched log, while the exporter is still inside its export; the test
                                     releases it afterwards. An exporter that panics: the export record, and the
                                     panic reaches no hook but the boundary's. Neither test waits a fixed time.
the cap, executable                  three collectors hold the bound from outside: one that reads the request and
                                     never answers (Answer::Hang), one that answers the head of a response and none
                                     of its body (Answer::Stall), and one that sends the body a byte every 20 ms
                                     (Answer::Drip). Each test asserts that the process ends within 2 s, with the
                                     stdout and the exit code of the same command with tracing off.
                                     no flake: the collector holds its connection until the test stops it, which
                                     the test does after the process has ended, so no test waits out a stall of a
                                     fixed length. A binary that waits for the collector never ends by itself and
                                     fails at the 30 s deadline of the runner. 2 s is twenty caps, and what fills
                                     it on a loaded machine is the start of the process and the reading of the
                                     trust store, nothing the binary waits for. No test asserts a bound near the
                                     cap: a loaded machine would fail it.
                                     what each shows: Drip is the case the outer bound alone holds — under the
                                     inner bound every read of the body succeeds, and the export lasts for hours.
                                     Stall costs two caps under the inner bound alone, which 2 s does not tell
                                     from one, so that test holds the outcome and the unit test above holds the
                                     bound. On a machine so loaded that the bytes of a drip arrive more than
                                     100 ms apart the inner bound ends a drip too, and the test then passes without
                                     telling the two apart; it never fails for it.
configured headers                   unit, crates/core/src/trace.rs: `settings` leaves a content-type and a header
                                     with an illegal value out of TraceConfig.headers and names the second in
                                     skipped_headers. Unit, crates/cli/src/trace.rs: one record for each skipped
                                     name, with the name and without the value. Executable: with a content-type
                                     and an illegal value configured, the one request carries the protobuf content
                                     type, the other configured headers, and not the skipped one.
redirects                            no test: the collector of the support package answers none (Open questions)
proxies                              no test: none sets a proxy variable (Open questions)
counted requests                     a test that counts the connections or the requests of one invocation hands it
                                     an endpoint whose path is the name of its tree, and asserts that every request
                                     its collector read was sent to that path followed by /v1/traces. It runs its
                                     invocation once and replaces no collector: a request at another target fails
                                     the test with the targets read.
a refused address                    a test of an endpoint nothing listens at holds a RefusedEndpoint of the support
                                     package until the child it handed the endpoint to has ended (test-harness.md,
                                     RefusedEndpoint)
TLS                                  no automated test completes a TLS handshake: the collector of the support
                                     package speaks none. An executable test names the plain collector with an
                                     `https` URL and asserts that a connection arrives and the export fails within
                                     the cap, which shows the scheme is served and bounded. The collector answers
                                     no handshake, so no certificate is judged in it.
no scenario                          the pinned environment of a scenario turns tracing off, and a collector is no
                                     dimension a scenario compares
no network                           every test addresses 127.0.0.1 alone; none resolves a name or reaches a real
                                     collector. A lookup that hangs is therefore under no test: the outer bound
                                     covers it because the wait does not ask what the export thread is doing.
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
                 span open         log.header_skipped for each skipped header; then
                                   the export thread is started; it ends the span, exports it, and shuts the
                                   provider down, and `finish` waits for its answer for the cap at most
                                     an answer in time    the thread is joined; on a failure, log.export_failed
                                     the cap expires      the thread is abandoned; log.export_failed
                 unavailable       log.unavailable, when a command was announced
                 otherwise         nothing
            -> the exit code, unchanged by anything `finish` did
exit      `main` returns the exit code and the process ends, waiting for no thread: an export thread abandoned at
          the cap, and the helper thread of its HTTP client, end with it
panic     the span is ended with `Internal` and exported like any other ending
          a panic on the export thread is a failed export: nothing on stderr without a logging switch, and the
          exit code is the ending's
SIGINT    undeferred: the handler exits the process; `finish` never runs and nothing is sent
          during the wait of `finish`: the handler exits the process as anywhere else; the request may be out
          deferred: the command returns Termination::Interrupted and the span is exported with `Interrupted`
          decision:parallel-workspaces/component-runtime/interrupt-trace-export
```

Export never changes the result: `finish` returns nothing, runs after the last byte of stdout and the diagnostic are
written, and cannot panic past the boundary; a collector that refuses, hangs, answers an error, answers the head of a
response and stalls, or drips its body, and a resolver that hangs, cost at most the cap, counted from the start of the
wait of `finish`, and leave stdout, the exit code, and an unswitched stderr as they were (reference:
`core/internal/otel_command_tracer.py:35,383-409`; `tests/test_cli_otel_process.py`, the hanging and refused endpoint
tests).

## Open questions

- These points were read in the sources of the releases and not yet run: that a header value handed with each `%`
  written `%25` arrives as `TraceConfig` holds it, and that a name is not decoded; that the disabled retry policy can be
  handed to the exporter with the features this contract enables; that explicit span limits, an explicit sampler, and a
  resource built from the empty one leave no SDK variable an effect; that `SSL_CERT_FILE` naming a file with one
  self-signed certificate lets the exporter be built while the host's store is not read; and what the blocking client's
  timeout covers, as the cap lists it. Four were not read: that each crate builds with default features off; that the
  unbatched processor exports on the thread that ends the span; that the SDK's span and its provider can be moved to
  another thread and ended there; and that the process ends at once when `main` returns while a thread it never joined
  is still blocked. The change that adds the crates confirms each against the release it takes; a point that does not
  hold goes back to engineering before the change merges.
- What reading the trust store costs an invocation was not measured: the store of the machine the sources were read on
  holds 243 certificates in 180 kB, read and parsed before the one request. The change that adds the crates reports the
  start-up time of a traced command beside an untraced one; engineering weighs it then.
- `SSL_CERT_FILE` and `SSL_CERT_DIR` were read for Linux. How the platform's verifier reads the store on another
  operating system, and whether the two variables apply there, was not read; engineering reads it when the binary is
  first built for one.
- What the client does on a redirect was read in the sources of reqwest 0.13.5 and tower-http 0.6.11 and not run: the
  collector of the support package answers no redirect, so no test has one. One point was not read to its end: that the
  client can send the body of the export a second time after a 307 or a 308. Engineering confirms both with the first
  test collector that answers a redirect.
- What the client does with a proxy variable was observed for `HTTP_PROXY` with an `http` endpoint. For `HTTPS_PROXY`,
  `ALL_PROXY`, `NO_PROXY`, the lower-case names, and an `https` endpoint it was read in the sources of reqwest 0.13.5
  and hyper-util 0.1.21 and not run, and no test names a proxy. Engineering confirms the rest with the first test that
  does.
