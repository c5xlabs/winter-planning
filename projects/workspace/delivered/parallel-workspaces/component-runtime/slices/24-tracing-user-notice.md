---
name: tracing-user-notice
depends-on:
  - clock-and-signal-sentences
code:
  - winter-workspace/crates/core/src/trace.rs::TraceConfig
  - winter-workspace/crates/cli/src/trace.rs::Tracer
  - winter-workspace/crates/core/src/parse.rs
  - winter-workspace/README.md
---

# Tracing is told to users, and the export keeps its own content type

Builds the rules for a configured header, the `header_skipped` record, and What a user is told of
[tracing.md](../spec/tracing.md), under `decision:parallel-workspaces/component-runtime/trace-header-precedence`,
`decision:parallel-workspaces/component-runtime/tracing-user-notice`, and `engineering:observability-isolation`. The
redirect behaviour the README states is the linked client's own, under
`decision:parallel-workspaces/component-runtime/trace-export-redirects`, and no code changes for it. It follows
`clock-and-signal-sentences`, which edits `context/architecture/structure.md` before it, and shares no file with
`shared-test-helpers`.

## Change

- `crates/core/src/trace.rs`: `settings` leaves two kinds of parsed header out of `TraceConfig.headers`, judged on the
  pairs after a repeated name took its last value: the name `content-type`, with nothing recorded, and a value that
  holds a byte below 0x20 other than the tab, or the byte 0x7F, whose name goes to the new field `skipped_headers` in
  configured order. `TraceLog` gains `header_skipped`, a Debug record of origin `trace` written only under a logging
  switch, whose text holds the name and never the value. The module's doc comment counts three records.
- That this byte rule is the one by which the exporter's HTTP crate refuses a value was read in `http` 1.5.0; which
  release `Cargo.lock` holds, and that its rule is the same, is to be confirmed by the builder.
- `crates/cli/src/trace.rs`: the open state keeps the skipped names, and `finish` writes one `header_skipped` for each,
  in order, before the record of the export, when a span is open and never otherwise. The doc comment of the exporter's
  construction no longer says the exporter drops a header or lets a `content-type` replace its own.
- Every literal `TraceConfig { .. }` gains the field; `grep -rn 'TraceConfig {' crates` lists them.
- `crates/cli/tests/trace.rs` gains the executable test of the Acceptance; its existing tests stay as they are.
- `crates/core/src/parse.rs`: the root command gains one line after its options, by the parser's `after_help`, that
  names `WINTER_OTEL_EXPORTER_OTLP_ENDPOINT` as the variable that makes a command send a trace span, and `README.md` as
  where what is sent is written. Groups and commands gain none. The line fits the fixed help width of 100 columns.
- Tests that read the root help: `crates/core/tests/root.rs`, `crates/cli/tests/root.rs`, and the unit tests of
  `parse.rs` compare the help with itself or search it for a word, and none pins its whole text, as the files were read;
  that `the_root_help_lists_exactly_the_groups_and_top_level_commands_of_the_registry` of `parse.rs` still passes with
  the added line is to be confirmed by the builder, who edits that test in `parse.rs` if it does not.
- `README.md` gains the section for users that [tracing.md](../spec/tracing.md), What a user is told, lists, item by
  item: tracing, the exit codes of the commands that exist, and the values of `WINTER_LOG_LEVEL`. It adds no scope
  token, and says "the installed `winter`" for the tool whose variable it shares.
- `context/architecture/structure.md`: the Modules row of the library's `trace.rs` counts three records about tracing.
- Files touched: `crates/core/src/{trace,parse}.rs`, `crates/cli/src/trace.rs`, `crates/cli/tests/{trace,root}.rs`,
  `README.md`, `context/architecture/structure.md`. `crates/core/src/lib.rs` is not touched: the facade re-exports
  `TraceConfig` and `TraceLog` already.
- `crates/dev/tests/architecture.rs`: the README is scanned for scope tokens, and the section holds none; the library's
  sources gain none of the forbidden tokens and no dependency, so no pin changes.
- Scenarios: none ([tracing.md](../spec/tracing.md), Test seams).

## Acceptance

- Unit tests in `crates/core/src/trace.rs`, `settings` over literal environments with the headers variable set to:
  - `content-type=text/plain,x-a=1`: `headers` is `[("x-a", "1")]` and `skipped_headers` is empty;
  - `Content-Type=a`, in capitals: the same, since the name is lower-cased before it is judged;
  - `x-a=1,x-bad=a%0Ab,x-b=2`: `headers` holds `x-a` and `x-b`, and `skipped_headers` is `["x-bad"]`; the same for
    `%7F`, and for `%00`;
  - `x-tab=a%09b` and `x-high=%C3%A9`: both are in `headers`, the tab and the two bytes above 0x7F kept;
  - `x-a=a%0Ab,x-a=ok`: the last value is judged, so `x-a` is sent with `ok` and nothing is skipped.
- Unit tests in `crates/cli/src/trace.rs`, through `Tracer::with_exporter`, with `skipped_headers` of `x-bad` and `x-c`:
  - under a switched log, stderr holds two Debug records of origin `trace` in that order, each with its name, and then
    the export record when the exporter answers an error; the span is handed to the exporter all the same;
  - under an unswitched log, and under a log switched at Error, stderr holds neither;
  - no command announced, settings `Unavailable`, and a builder that answers an error: no such record.
- `crates/cli/tests/trace.rs`, the built binary with a `Collector` as the existing tests of the file set it up, and
  `WINTER_OTEL_EXPORTER_OTLP_HEADERS` of `content-type=text/plain,x-bad=s3cr3t%0Aprobe,x-plain=b`:
  - one request arrives; its `content-type` is `application/x-protobuf` alone, it carries `x-plain` with `b` and no
    `x-bad`, and its body reads through `decode_export`; exit code, stdout, and stderr are those of tracing off;
  - with `-v`, stderr holds one record of origin `trace` that names `x-bad`, and neither `s3cr3t` nor `probe` occurs in
    it; no record names `content-type`.
- `winter-workspace --help`, run by `crates/cli/tests/root.rs` in an isolated tree, prints a line that holds
  `WINTER_OTEL_EXPORTER_OTLP_ENDPOINT` and `README.md`; the stderr of a bare root holds the same line; and the help of
  `ws` and of `ws index` holds neither word.
- `grep -c 'WINTER_OTEL_EXPORTER_OTLP_ENDPOINT\|OTEL_SDK_DISABLED\|WINTER_LOG_LEVEL' README.md` is at least 3, and the
  section states each item of [tracing.md](../spec/tracing.md), What a user is told, in the README's own words: read
  against that list, none is missing and none says more than the spec.
- `cargo test -p winter-workspace-dev` exits 0: the README holds no scope token.
- `grep -rn 'two records\|two log records' context crates/core/src/trace.rs` prints nothing.
- `grep -n 'drops a header' crates/cli/src/trace.rs` prints nothing.
- `git diff --stat` of the change names no manifest and no file under `crates/test-support` or `crates/core/tests`.
- `cargo dev gate slice` exits 0.

## Out of scope

- A test of a redirect: the collector of the support package answers none ([tracing.md](../spec/tracing.md), Open
  questions).
- A record for an ignored `content-type`: the spec writes none.
- Help of a group or a command, a manual of the commands, and install steps in the README.
- A switch that turns redirects off: `decision:parallel-workspaces/component-runtime/trace-export-redirects`.
