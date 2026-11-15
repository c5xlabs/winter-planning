---
name: command-span-record
depends-on:
  - runtime-documents
code:
  - winter-workspace/crates/core/src/termination.rs::Termination
  - winter-workspace/crates/core/src/lib.rs
  - winter-workspace/crates/test-support/src/lib.rs
---

# The command span's settings and vocabulary, and the collector of the tests

Builds the part of [tracing.md](../spec/tracing.md) that needs no tracing crate: Domain as far as `settings` reads it,
`TraceSettings`, `TraceConfig`, `settings`, and `span_name` of The library's part, and `Termination::error_type`; and
the decoder and the loopback collector of [test-harness.md](../spec/test-harness.md), Interfaces, under
`engineering:observability-isolation`, `engineering:testable-boundaries`,
`decision:parallel-workspaces/component-runtime/span-error-type`, and
`decision:parallel-workspaces/component-runtime/trace-exporter`. Nothing is sent and no behaviour of the binary changes.
It follows `runtime-documents` and is built before `command-span-export`; it runs beside no other change.

## Change

- `crates/core/src/trace.rs` exists with `TraceSettings`, `TraceConfig`, `settings`, and `span_name` as the spec gives
  them. It takes the instant as an argument, reads the environment only through `ProcessEnv`, and names no tracing
  crate. The tracer port and the two records are not built: nothing calls them yet.
- `termination.rs`: `Termination::error_type` exists, an exhaustive match over the variants built so far with the
  literals of the spec's vocabulary.
- `lib.rs`: the facade re-exports `settings`, `span_name`, `TraceConfig`, and `TraceSettings`.
- `crates/test-support/src/otlp.rs` exists with `ReceivedExport`, `ReceivedSpan`, `decode_export`, `ReceivedRequest`,
  `Answer`, and `Collector` as [test-harness.md](../spec/test-harness.md), Interfaces, gives them. The decoder is
  written by hand from the field table there, and the collector listens on 127.0.0.1 alone.
- Contributor documents, each a sentence this change makes false or a rule it adds:
  - `context/architecture/structure.md`: the `trace.rs` row of the library is added, and the support package's role
    names the collector.
  - `context/architecture/changes.md`, An error: a new `Termination` variant adds its name to `error_type`, and a name
    is never renamed.
  - `context/standards/testing.md`: what the collector is, that it is the one socket a test opens, and that it binds the
    loopback address alone.
  - `AGENTS.md`, the test rule, gains the loopback exception as
    [contributor-context.md](../spec/contributor-context.md), Entry point, words it.
- Files touched: `crates/core/src/trace.rs` (new), `crates/core/src/{termination,lib}.rs`,
  `crates/test-support/src/otlp.rs` (new), `crates/test-support/tests/otlp.rs` (new), `crates/test-support/src/lib.rs`,
  `context/architecture/{structure,changes}.md`, `context/standards/testing.md`, `AGENTS.md`.
- No manifest changes and no crate to fetch: the decoder and the collector use the standard library alone.
- `crates/dev/tests/architecture.rs` pins nothing this change alters. `trace.rs` spells none of the forbidden tokens;
  `otlp.rs` holds neither the reference variable's name nor the word the support package never spells; the reworded test
  rule of `AGENTS.md` holds no scope token.
- Scenarios: none ([tracing.md](../spec/tracing.md), Test seams).

## Acceptance

- Unit tests of `settings`, each over a literal environment and a fixed instant:
  - every row of When tracing is on: a blank endpoint, the kill switch in three letter cases and with blanks around it,
    the values `1` and `yes` that do not switch it, a generic `OTEL_EXPORTER_OTLP_ENDPOINT` alone, which leaves tracing
    off, and an endpoint whose value is not UTF-8, which gives `Unavailable`;
  - the endpoint: a value with no, one, and several trailing slashes gives the same string ending in `/v1/traces`; an
    `http` and an `https` value both give `On`, each kept as given;
  - headers: two items, a percent-encoded value, an upper-case name lower-cased, and an item with no `=` skipped while
    the next applies; a value that is not UTF-8 gives no headers;
  - `TRACEPARENT`: a value is carried as given, sampled or not; absent, empty, and not UTF-8 each give `None`;
  - `WINTER_LAUNCH_TIME`: an integer, a `.` fraction, a `,` fraction, a fraction of more than nine digits, zero, a value
    later than the instant, and text;
  - the resource: `OTEL_SERVICE_NAME` over `service.name` of `OTEL_RESOURCE_ATTRIBUTES` over `winter`, with the other
    entries kept and `service.name` first.
- `span_name(["ws", "index"])` is `winter ws index`.
- A unit test of `error_type`: for one value of every `Termination` variant the result is `None` exactly when
  `exit_code()` is 0, and the names returned are the literal list of the spec's vocabulary that the variants built so
  far cover.
- Unit tests of `decode_export`, each over a literal byte string the test holds, built by hand from the field table: a
  root span with one attribute and no status; a child span with two attributes and status code 2 with an empty message;
  a message with fields the table does not name, which is read; a truncated message, which is an error.
- Tests of `Collector`, in `crates/test-support/tests/otlp.rs`, each with a client of the test's own over
  `std::net::TcpStream`:
  - `Answer::Ok`: a `POST` with a `Content-Length` body is answered with a success status, and `requests()` holds its
    method, target, headers, and body;
  - `Answer::Status(500)`: the request is recorded and the client reads that status;
  - `Answer::Hang`: the request is recorded and no answer arrives before the client's own read deadline;
  - a client that connects and sends bytes that are no HTTP request raises `connections()` and leaves `requests()`
    empty;
  - nothing accepts at `refused_endpoint()`;
  - dropping the collector ends its thread and frees its port.
- `git diff --stat` of the change names no `Cargo.toml` and not `Cargo.lock`.
- `cargo dev gate full` exits 0.

## Out of scope

- The tracer port, the two records, the binary's tracer, the crates, and the export: `command-span-export`.
- Whether a `TRACEPARENT` is valid or sampled: the SDK's propagator and sampler decide it, in `command-span-export`.
