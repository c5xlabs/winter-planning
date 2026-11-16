---
name: command-span-export
depends-on:
  - command-span-record
code:
  - winter-workspace/crates/cli/src/main.rs::conclude
  - winter-workspace/crates/cli/Cargo.toml
  - winter-workspace/crates/core/src/process.rs::Invocation
  - winter-workspace/crates/core/src/trace.rs
  - winter-workspace/crates/test-support/src/otlp.rs::Answer
---

# The command span is opened, ended, and exported once

Builds the tracer port, the two records, The binary's part, and Flows of [tracing.md](../spec/tracing.md), with the
fourth parameter of `conclude` and `Invocation.tracer` of [runtime.md](../spec/runtime.md), under
`engineering:observability-isolation`, `decision:observability-shutdown`,
`decision:parallel-workspaces/component-runtime/trace-export-transport`,
`decision:parallel-workspaces/component-runtime/trace-exporter`,
`decision:parallel-workspaces/component-runtime/trace-export-total-bound`,
`decision:parallel-workspaces/component-runtime/interrupt-trace-export`, and
`decision:parallel-workspaces/component-runtime/trace-export-trust-roots`: the exporter verifies an `https` endpoint
against the operating system's trust store, and the binary bundles no roots. It follows `command-span-record`, is the
last change of the epic, and runs beside no other change.

## Change

- `crates/cli/Cargo.toml` names `opentelemetry`, `opentelemetry_sdk`, and `opentelemetry-otlp`, each at `0.33.1` with
  `default-features = false`: `opentelemetry` and `opentelemetry_sdk` with the feature `trace`, and `opentelemetry-otlp`
  with `trace`, `http-proto`, `reqwest-blocking-client`, and `reqwest-rustls`, as
  [repository-layout.md](../spec/repository-layout.md), The Cargo workspace, writes the three lines. The manifest names
  no other new crate: the HTTP client, the TLS stack, and `tokio` arrive through the exporter alone. `reqwest-rustls` is
  the TLS feature, under `decision:parallel-workspaces/component-runtime/trace-export-trust-roots`: it verifies through
  the platform's trust store. `Cargo.lock` gains the three crates and everything they pull in, `tokio` and `aws-lc-sys`
  among them: 161 packages as the three lines resolved when they were read, 27 with a build script and 12 procedural
  macros, with no package changed or removed. A resolution that differs is reported with the difference and stops
  nothing. The build compiles bundled C, so it needs a C compiler; whether `aws-lc-sys` needs CMake beside it is found
  by building. The build of this change fetches from crates.io; after it, `cargo fetch --locked` is again the only step
  that needs the network.
- Before it writes anything else, the change confirms against the releases it takes each point the Open questions of
  [tracing.md](../spec/tracing.md) mark as read and not run, or as not read, and stops at the first that does not hold,
  which goes back to engineering with nothing merged:
  - that each of the three crates builds on toolchain 1.99.1 with default features off;
  - that the unbatched processor calls the exporter on the thread that ends the span;
  - that the SDK's span and its provider can be moved to a thread other than the one that opened the span, and the span
    ended there;
  - that the process ends at once when `main` returns its exit code while a thread it never joined is still blocked;
  - that the blocking client's timeout covers what the cap of [tracing.md](../spec/tracing.md) lists for the inner
    bound, and no more;
  - that with `SSL_CERT_FILE` naming a file that holds the one certificate of `crates/cli/tests/data/trust-root.pem`,
    and `SSL_CERT_DIR` unset, the exporter is built and a request reaches a plain collector; and that with
    `SSL_CERT_FILE` naming an empty file the exporter is not built. The two together show that the variable governs the
    reading, which is what lets every test of the real exporter stay inside its tree;
  - that a header value handed to the exporter with each `%` written `%25` is sent as `TraceConfig` holds it, and that a
    header name is sent as handed;
  - that the exporter takes the disabled retry policy with the features of the three lines;
  - that with explicit span limits, an explicit sampler, and a resource built from the empty one, no `OTEL_*` variable
    alters the span, and that settings `Off` and `Unavailable` construct nothing whatever the environment holds.
- `crates/core/src/trace.rs`: `CommandTracer`, `NoTrace`, and `TraceLog` exist as the spec gives them; the two records
  go through `diagnostics::record`.
- `crates/core/src/process.rs`: `Invocation` carries `tracer`; step 2 of `run_with` announces the command once, after
  the Debug record, at `Stage::Command` whatever the outcome; `run_in_memory` passes `NoTrace`. `ProcessEnv::names()`
  exists as [repository-layout.md](../spec/repository-layout.md) gives it.
- `crates/core/src/lib.rs`: the facade re-exports `CommandTracer`, `NoTrace`, and `TraceLog` beside `settings`,
  `span_name`, `TraceConfig`, and `TraceSettings`, which it re-exports already.
- `crates/cli/src/trace.rs` exists with `Tracer`, `Tracer::new`, `Tracer::with_exporter`, `finish`, and
  `hide_generic_exporter_variables`. Its one `unsafe` block is the removal of the variables, with the reason the spec
  gives. It names no asynchronous runtime and starts none: the runtime of the exporter's HTTP client is the client's
  own. The exporter is built as [tracing.md](../spec/tracing.md), The binary's part, construction, gives it: the
  protocol named explicitly, each header value with `%` written `%25`, the name `user-agent` handed as `User-Agent`, the
  timeout of 100 ms, and the disabled retry policy; the provider with the explicit sampler, explicit span limits, and a
  resource built from the empty one. A trust store that yields no certificate fails the building of the exporter and
  turns the tracer unavailable, like any other such failure.
- `finish` ends an open span as [tracing.md](../spec/tracing.md), The binary's part, gives it under finish, the export
  thread, the cap, and at exit:
  - it sets the error type and reads the end instant on the ending thread, then starts the thread `trace-export`, which
    owns the span and the provider, ends the span at that instant, reads the kept result, shuts the provider down, drops
    both, and sends its answer as the last thing it does;
  - the body of that thread arms the panic slot of its own thread, catches its unwind, and answers a panic as a failed
    export; `arm` and `disarm` of `main.rs` are what it calls, and they stay private to the binary;
  - `finish` waits for the answer with a timeout of 100 ms from the start of the wait. On an answer it joins the thread
    and writes the record the answer asks for. On expiry it drops the handle without joining and writes the export
    record with the cause `the export exceeded its time of 100 ms`;
  - neither `finish` after it started the thread, nor `main`, holds the provider, the exporter, or the HTTP client, and
    after a failed start `finish` forgets the span and the provider instead of dropping them;
  - `main` still returns `ExitCode` and joins nothing: nothing in the binary waits for the export thread after an
    expiry.
- `crates/cli/src/main.rs`: `main` reads `settings`, hides the generic exporter variables when they are `On`, builds the
  tracer, and passes it; `conclude` has its fourth parameter and step 5.
- `crates/test-support/src/otlp.rs`: `Answer` gains `Stall` and `Drip` as [test-harness.md](../spec/test-harness.md),
  Interfaces, gives them. Both record the request and write the head of a 200 with `content-length: 1048576`; `Stall`
  then holds the connection silent, and `Drip` writes one byte every 20 ms from the listener thread. Stopping or
  dropping the collector ends either at once.
- The scaffold passes a recording tracer, and `Ended` gains `announced`.
- `crates/cli/tests/data/trust-root.pem` holds one self-signed certificate made for the tests, whose key is discarded
  and committed nowhere. A test compiles its text in and writes it under `<ROOT>/tmp` of its tree; none reads the
  committed file when it runs. The change's report says how the certificate was made.
- The real exporter is built in no test process: the tests of `crates/cli/src/trace.rs` go through
  `Tracer::with_exporter`, and those that need the real exporter are executable tests, or the one child test of the
  interrupt ([tracing.md](../spec/tracing.md), Test seams).
- Contributor documents, each a sentence this change makes false or a rule it adds:
  - `AGENTS.md`, the library-boundary rule, names sockets, as [contributor-context.md](../spec/contributor-context.md),
    Entry point, words it. The test rule gains no exception.
  - `context/architecture/structure.md`: the Modules table gains the row of the binary's `trace.rs`, which names the
    export thread and that `main` waits for no thread after the cap, and the library's row names the port; The library
    and the binary counts three files of the binary, hands `conclude` four things, lists the tracer among what an
    `Invocation` carries, and names `trace.rs` wherever it says which files and which test modules of the binary package
    name the process environment or start a fixture; the dependency table gains one row for each of the three crates,
    with its features and its user, and the row of the binary's dev-dependency names the test module of `src/trace.rs`.
  - `context/standards/rust.md`: `unsafe` appears in `crates/cli/src/interrupt.rs` and `crates/cli/src/trace.rs` alone;
    Output and environment access says the library opens no socket, reads no clock, and names no tracing crate.
  - `context/standards/testing.md`: a test wanting a span clears the kill switch of the pinned environment and sets
    `SSL_CERT_FILE` to the certificate file it wrote in its tree; the real exporter is built only in a child started
    under the pinned environment, the built binary by preference, and never in a test process; the unit tier's exception
    for the binary package names the tracer under the interrupt handler beside the handler, with the collector the
    parent holds for it; a test of a bound in time holds its peer until the subject has ended and waits out no stall of
    a fixed length.
  - `context/tooling/development.md`: the prerequisites name a C compiler, which the build of the binary needs for the
    cryptography of the TLS stack, and anything else that build needed on the machine of this change; the sentence that
    names bootstrap as the only step that needs the network stays. No prerequisite names a certificate store: the tests
    bring their own.
- The backlog's dependency lines, [repository-layout.md](../spec/repository-layout.md), The Cargo workspace, are
  corrected where a version or a feature name this change took differs from them.
- Files touched: `crates/cli/Cargo.toml`, `Cargo.lock`, `crates/cli/src/trace.rs` (new), `crates/cli/src/main.rs`,
  `crates/core/src/{trace,process,lib}.rs`, `crates/core/tests/support/in_process.rs`, `crates/core/tests/root.rs`,
  `crates/cli/tests/trace.rs` (new), `crates/cli/tests/data/trust-root.pem` (new), `crates/test-support/src/otlp.rs`,
  `crates/test-support/tests/otlp.rs`, `AGENTS.md`, `context/architecture/structure.md`,
  `context/standards/{rust,testing}.md`, `context/tooling/development.md`. Nothing else of `crates/test-support/` is
  touched.
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
  - the exporter is called on a thread other than the one that called `finish`, and `finish` has returned only after
    that thread ended: the test reads the thread's id inside the export and finds the handed span once `finish` is back;
  - an exporter that does not return before the test releases it, or before 30 s: `finish` returns while the exporter is
    still inside its export, having written nothing under an unswitched log and, under a switched one, the one record of
    origin `trace` whose cause is `the export exceeded its time of 100 ms`. The test releases the exporter after its
    assertions, so it waits out nothing; an implementation that waits for the exporter fails at the 30 s;
  - an exporter that panics inside its export, with the boundary's hook installed over one that notes the panicking
    thread, as the tests of `conclude` install it: `finish` returns, writes nothing under an unswitched log and one
    record of origin `trace` under a switched one, and the panic was handed to no hook but the boundary's;
  - a builder that answers an error, and settings `Unavailable`: nothing is exported; one record under a switched log at
    Debug, none otherwise, and none when no command was announced.
- A unit test of `hide_generic_exporter_variables`, in a child test of the same test binary by the fixture rule of the
  unit tier's exceptions: with `OTEL_EXPORTER_OTLP_ENDPOINT` and `OTEL_EXPORTER_OTLP_HEADERS` in the child's
  environment, after the call the process environment holds neither and the snapshot holds both.
- Component tests, through the scaffold: `announced` is `["ws", "index"]` for `ws index NAME` and for
  `ws index --bogus`, `["ws"]` for `ws --help` and for `ws nope`, and `None` for `--version`, `--help`, `--bogus`,
  `nope`, `-v -v ws index NAME`, and no arguments.
- `crates/test-support/tests/otlp.rs`, each with a plain `TcpStream` as the client and a read timeout of its own:
  - `Answer::Stall`: after a request, the head of a 200 arrives with `content-length: 1048576`, and a read of 300 ms
    after it yields no byte; `requests()` then answers at once with the one request;
  - `Answer::Drip`: after a request, the same head arrives and at least three bytes of the body follow within 2 s, no
    two of them in one write; dropping the collector returns at once, and the client's next read finds the connection
    closed.
- `crates/cli/tests/trace.rs`, the built binary in an isolated tree with a `Collector`, the kill switch cleared, and
  `SSL_CERT_FILE` naming the certificate file the test wrote in its tree, unless the test says otherwise:
  - `ws index NAME` with a sampled `TRACEPARENT` and a `WINTER_LAUNCH_TIME` written with a comma: exit 0, stderr empty,
    one request; its span is named `winter ws index`, has the trace id and the parent of the `TRACEPARENT`, starts at
    the launch time, and carries only `winter.command`; the resource's `service.name` is `winter`;
  - the same request has method `POST`, a target ending in `/v1/traces`, and `content-type: application/x-protobuf`, and
    its body reads through `decode_export`;
  - `WINTER_OTEL_EXPORTER_OTLP_HEADERS` of `x-probe=a%2541,x-plain=b`: the request carries `x-probe` with the value
    `a%41` and `x-plain` with `b`. Without the escape the first arrives as `aA`;
  - `ws index` with no name: exit 2; the span's attributes are exactly `winter.command` and `error.type` of `Usage`, and
    its status is code 2 with an empty message;
  - the same command with `OTEL_SPAN_ATTRIBUTE_COUNT_LIMIT` of `0`, and again of `1`, in the child's environment: the
    span carries the same two attributes. With `OTEL_TRACES_SAMPLER` of `always_off`: one request still arrives;
  - `--version`, `nope`, and no arguments: no connection;
  - the kill switch left at `true`, and a blank endpoint: no connection; an unsampled `TRACEPARENT`: no request;
  - `Answer::Hang`, `Answer::Status(500)`, and a refused endpoint: stdout, stderr, and the exit code are those of the
    same command with tracing off, and the process ends within 2 s;
  - `Answer::Stall`: stdout, stderr, and the exit code are those of the same command with tracing off, the process ends
    within 2 s, and afterwards the collector holds the one request; with `-v`, stderr holds the export record of origin
    `trace`. The collector is stopped only after the process has ended;
  - `Answer::Drip`: the same four assertions, and the same record under `-v`. This is the behavioural check of the bound
    the binary holds itself, and no search of the sources stands in for it: with the exporter's timeout alone every read
    of the dripped body succeeds, the process does not end, and the test fails at the 30 s deadline of the runner. The 2
    s are twenty caps, taken up on a loaded machine by the start of the process and the reading of the trust store; the
    drip lasts until the test stops the collector, so nothing here waits a fixed time;
  - `-v` with `Answer::Ok`: stderr holds no record of origin `trace`; `-v` with `Answer::Status(500)`, with
    `Answer::Hang`, and with a refused endpoint: stderr holds the export record of origin `trace`;
  - `Answer::Status(503)`, an answer the exporter retries by default, and `Answer::Hang`: once the process has ended,
    `connections()` counts exactly one;
  - `SSL_CERT_FILE` naming an empty file of the tree, with a plain `http` collector: exit 0, stdout that of the same
    command with tracing off, stderr empty, and `connections()` counts none; with `-v`, stderr holds the one Debug
    record of origin `trace` that says tracing is unavailable, and no export record;
  - the `Collector`'s own endpoint with its scheme written `https`, the address still 127.0.0.1 and no certificate of
    the collector anywhere: `connections()` counts one, no request is recorded, the exit code and stdout are unchanged,
    stderr is empty without `-v`, and the process ends within 2 s. This is the one test of `https`; none completes a
    handshake;
  - `OTEL_EXPORTER_OTLP_ENDPOINT` and `OTEL_EXPORTER_OTLP_TRACES_ENDPOINT` naming a second collector beside the Winter
    endpoint: the first collector holds the one request and the second sees no connection; the same two variables with
    no Winter endpoint: no connection at either.
- A unit test in `crates/cli/src/trace.rs` that starts a child test of the same test binary through `invoke`, with
  `SSL_CERT_FILE` in the child's environment naming the certificate file the parent wrote in its tree: the child
  installs the interrupt handler, announces a command to a `Tracer` whose endpoint is the parent's `Collector`, writes a
  ready line, and raises SIGINT on itself; the child exits 1 and the collector holds no request.
- `grep -n 'RetryPolicy::disabled' crates/cli/src/trace.rs` prints a line. The count of connections above holds the
  behaviour, and this line holds the call: as the release was read, the cap alone already leaves a retry no room.
- No test of `crates/cli/src/trace.rs` calls `Tracer::new` outside a child test, and `grep -rn 'SSL_CERT_DIR' crates`
  prints nothing.
- `grep -rn 'unsafe' crates/cli/src` prints lines of `interrupt.rs` and `trace.rs` only, each block preceded by a
  comment; `grep -rn 'opentelemetry' crates/core crates/test-support crates/dev` prints nothing.
- `grep -rnE 'tokio|async[-_]std|block_on' crates/cli/src crates/core/src` prints nothing: the sources of the binary and
  the library name no asynchronous runtime and start none.
- `git diff --stat` of the change names `crates/cli/Cargo.toml` and `Cargo.lock`, and no other manifest. The diff of
  `crates/cli/Cargo.toml` adds the three lines of [repository-layout.md](../spec/repository-layout.md), The Cargo
  workspace, and no other dependency; `Cargo.lock` holds the three crates at 0.33.1.
- The change's report lists every package `Cargo.lock` gained, by name and version, as the diff of the lock gives them,
  with their count beside the 161 read before, and gives the start-up time of a traced command beside an untraced one.
- After the change, `cargo fetch --locked` with the network exits 0, and then `cargo dev gate full` with no network
  exits 0. No acceptance step runs `cargo fetch` with `--offline`: it would ask for crates of targets the host never
  builds.

## Out of scope

- Spans of single operations, `winter.env`, trace context in a child's environment, and the dashboard's spans:
  [tracing.md](../spec/tracing.md), System, names the owner of each.
- A collector whose certificate chains to a private authority: reachable over `https` when that authority is in the
  machine's trust store, under `decision:parallel-workspaces/component-runtime/trace-export-trust-roots`, and installed
  by no test. A completed TLS handshake in a test: [tracing.md](../spec/tracing.md), Test seams.
- A test of a host-name lookup that hangs: no test resolves a name. The wait of `finish` bounds it with everything else
  the export thread does ([tracing.md](../spec/tracing.md), The binary's part, the cap).
- A configured header the exporter drops or lets replace one of its own, and a collector that answers a redirect:
  [tracing.md](../spec/tracing.md), Open questions, each awaiting the product owner.
- A span for a deferred interrupt: built with the first command that returns `Termination::Interrupted`.
