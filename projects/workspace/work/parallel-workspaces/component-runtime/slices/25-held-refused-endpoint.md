---
name: held-refused-endpoint
depends-on:
  - tracing-user-notice
code:
  - winter-workspace/crates/test-support/src/otlp.rs::Collector
  - winter-workspace/crates/test-support/src/lib.rs
  - winter-workspace/crates/cli/tests/trace.rs::Fixture
  - winter-workspace/context/standards/testing.md
---

# The refused address of the tests stays owned while it is used

Replaces `Collector::refused_endpoint`, which answers the address of a port it bound and released, with
`RefusedEndpoint` of [test-harness.md](../spec/test-harness.md), Interfaces: a value that owns a bound socket which
never listens, so the address is refused for as long as the value lives and the system gives its port to no collector,
under `engineering:testable-boundaries`. A released port can be handed to a `Collector` another test starts, in the same
process or in another one, and the invocation aimed at the refused address then exports to that collector; the trace
tests fail intermittently for it. No source under `crates/*/src` of the library or the binary changes. It follows
`tracing-user-notice`, which gave `crates/cli/tests/trace.rs` the helper this change reduces.

## Change

- `crates/test-support/src/otlp.rs` gains `RefusedEndpoint` with `hold` and `endpoint`, built as
  [test-harness.md](../spec/test-harness.md), RefusedEndpoint, gives it: a socket made through `libc`, bound to
  127.0.0.1 on a port the system chooses, never listening, with close-on-exec and without `SO_REUSEADDR`, held in an
  `OwnedFd` from the moment it exists. Each `unsafe` block carries a comment that states why it is sound.
  `Collector::refused_endpoint` is removed, with no alias left behind. `crates/test-support/src/lib.rs` re-exports
  `RefusedEndpoint` by name beside `Collector`.
- The lifetime rule of the spec binds every caller: the value is kept until every child that was handed its endpoint has
  ended. The callers of the removed function, as `grep -rn 'refused_endpoint' crates` lists them, are two:
  - `crates/cli/tests/trace.rs`, `failing_endpoints`: each entry carries what keeps its address as it is, the
    `Collector` or the `RefusedEndpoint`, in place of the `Option<Collector>`. Its two callers,
    `a_collector_that_is_silent_failing_or_away_changes_nothing_and_does_not_keep_the_process` and
    `an_export_that_fails_is_one_error_record_about_tracing_under_the_verbose_flag`, bind that holder for the whole body
    of their loop, as they bind the collector today, so it outlives the invocation.
  - `crates/test-support/tests/otlp.rs`, `nothing_accepts_at_the_refused_endpoint`: replaced by the tests of the
    Acceptance.
- `crates/cli/tests/trace.rs`, `Fixture::alone`: the loop that sets a collector aside and runs the invocation again, and
  `COLLECTORS_TRIED`, are removed. They compensated for the released port alone: with the address held, no invocation of
  these tests is aimed at a port a collector can be given, and a second run of the invocation would now hide a request
  that should fail the test. The endpoint path of its own and the check on it stay, as [tracing.md](../spec/tracing.md),
  Test seams, counted requests, gives them: the invocation runs once, and a request the collector read at another target
  than `/<name of the tree>/v1/traces` fails the test with the targets read. That check also holds that the binary
  appends the traces path to an endpoint that has a path. The doc comment of `alone` says this and no longer names a
  released port. `connections_before` and every assertion of the callers of `alone` stay as they are.
- Every other test of `crates/cli/tests/trace.rs` and the unit tests of `crates/cli/src/trace.rs` stay as they are: none
  calls the removed function, and each was exposed only through the port another test released.
- `crates/test-support/tests/otlp.rs`: the module comment, which says every client connects to a collector and to
  nothing else, names the refused address and the listeners of the new tests as well.
- `context/standards/testing.md`:
  - The trace collector, the list of sockets: the bound socket of a `RefusedEndpoint`, which never listens, in place of
    the port bound and released; the connection attempt to the address of a `RefusedEndpoint` that is still held; and,
    among the listeners of the support package's own tests, the one whose bind to a held port fails and the one that
    shows the port of a dropped `RefusedEndpoint` free.
  - What the collector gives a test: the bullet on `Collector::refused_endpoint` is replaced by one on
    `RefusedEndpoint`. It states the lifetime rule and drops the sentence that another process may take the port before
    the test connects.
  - A test that wants a span: the endpoint is that of a `Collector` the test started or of a `RefusedEndpoint` it holds.
- Files touched: `crates/test-support/src/{otlp,lib}.rs`, `crates/test-support/tests/otlp.rs`,
  `crates/cli/tests/trace.rs`, `context/standards/testing.md`. No manifest: the support package names `libc` already.
- `crates/dev/tests/architecture.rs`: the support package gains no dependency and neither forbidden name, and the
  document keeps its anchors, so no pin changes; that no pin holds a sentence the document loses is to be confirmed by
  the builder with `cargo test -p winter-workspace-dev`.
- `context/standards/rust.md` limits `unsafe` in the library and the binary and is not touched: the support package
  holds `unsafe` in `process.rs` already.
- Scenarios: none.

## Acceptance

- The three points [test-harness.md](../spec/test-harness.md), RefusedEndpoint, marks as resting on the system are
  confirmed by the four tests below on the machine that builds the change. When a connection to the held address is not
  refused with `ConnectionRefused`, or a listener can be bound to a held port, the builder stops and hands the result
  back to engineering: no retry, no sleep, no replaced collector, and no released port is put in its place.
- `crates/test-support/tests/otlp.rs`, each client a plain `TcpStream` of the test:
  - with one `RefusedEndpoint` held, three connections in a row to its address each fail with
    `ErrorKind::ConnectionRefused`, and so does a fourth made after a `Collector` was started and dropped in between;
  - 64 values held at once have 64 different ports;
  - with one value held, `TcpListener::bind` to its port on 127.0.0.1 fails with `ErrorKind::AddrInUse`; after the value
    is dropped the same bind succeeds;
  - with one value held, 64 collectors started one after another and all kept running have 64 ports of which none is the
    held one, and a connection to the held address is still refused once they are dropped.
- `grep -rn 'refused_endpoint' crates context` prints nothing.
- `grep -n 'COLLECTORS_TRIED\|bound and released' crates/cli/tests/trace.rs crates/test-support/src/otlp.rs` prints
  nothing.
- `grep -n 'released' context/standards/testing.md` prints nothing.
- `grep -c 'RefusedEndpoint' crates/test-support/src/lib.rs` prints 1.
- With the traces path of the binary changed to another text, a mutation the builder reverts, each test that calls
  `Fixture::alone` fails on the first run of its invocation, and the failure prints the target the collector read.
- `cargo test -p winter-workspace-test-support --test otlp` and `cargo test -p winter-workspace --test trace` exit 0,
  and the second runs the 22 tests it ran before the change.
- The repeat run below prints nothing: 30 rounds of two copies of the test file at once, 60 runs. It is not made
  heavier: more than two copies at once, or more runs, exhaust the loopback ports of the machine.

  ```sh
  cargo test -p winter-workspace --test trace --no-run
  for round in $(seq 1 30); do
    cargo test -p winter-workspace --test trace >/dev/null 2>&1 & first=$!
    cargo test -p winter-workspace --test trace >/dev/null 2>&1 & second=$!
    wait "$first" || echo "round $round: a run failed"
    wait "$second" || echo "round $round: a run failed"
  done
  ```

- `git diff --stat` of the change names the five files of Files touched and no other.
- `cargo dev gate slice` exits 0.

## Out of scope

- A test that tells the disabled retry policy from the default one at the shipped timeout: no spec asks for it.
- A collector that answers a redirect, and a test that names a proxy: [tracing.md](../spec/tracing.md), Open questions.
- An own endpoint path for the tests that do not count: with the address held they receive no request from elsewhere.
- `RefusedEndpoint` on an operating system other than Linux: [test-harness.md](../spec/test-harness.md), Open questions.
