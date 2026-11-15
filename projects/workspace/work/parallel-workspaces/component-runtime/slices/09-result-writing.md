---
name: result-writing
depends-on:
  - contributor-context
  - epic:rust-adoption/parity-validation
code:
  - winter-workspace/crates/core/src/commands/ws/index.rs::run
  - winter-workspace/crates/core/src/commands/repo/remove.rs::run
  - winter-workspace/crates/core/src/parse.rs::written
---

# One owner for writing a result, and the closed-stdout ending

Builds `crates/core/src/output.rs` as [command-registration.md](../spec/command-registration.md), Writing a result,
gives it, and `Termination::BrokenPipe` as [results-and-exit-codes.md](../spec/results-and-exit-codes.md), Domain and
Internal failures ("closed stdout pipe: exit 1; nothing is written"), under `engineering:typed-outcomes`. The two
commands of `epic:rust-adoption/parity-validation` and the parser's help and version path stop classifying a failed
write themselves and return what the module returns.

## Change

- `crates/core/src/output.rs` exists with exactly the three functions of the spec, `line`, `text`, and `document`, each
  taking the handle and returning a `Termination`. One private function of the module maps an `std::io::Result<()>` to
  the ending, as the spec's "how a write ends" lists it: `Success`; `Termination::BrokenPipe` for an error whose kind is
  `std::io::ErrorKind::BrokenPipe`; `Termination::Internal(InternalError::Unexpected { .. })` for every other error,
  with the backtrace captured there. `document` encodes into a buffer with `serde_json::to_writer`, appends the newline,
  and hands the line to the handle in one `write_all`; an encoding error ends as the internal failure.
- `Termination` gains `BrokenPipe`: `exit_code()` is 1 and `diagnostic()` is `None`. The variant carries nothing.
- `ws index` and `repo remove` end `run` with `output::document(cx.out, &document(..))` under `--json` and
  `output::line(cx.out, &line(..))` otherwise. Each keeps its `document` and `line` functions, which build the value and
  the text; its `write_document` and `write_line` functions, and its imports of `InternalError`, `Backtrace`,
  `std::error::Error`, and `std::io::Write`, are gone.
- `parse.rs`: the help a flag asks for goes through `output::text` and the version line through `output::line`; its
  `written` function is gone. Its `internal` function stays for the one caller that is no write, a registry the tree
  cannot be built from.
- The process boundary is unchanged: `crates/cli/src/main.rs` already ends with exit 1 on a failed flush and writes a
  diagnostic only when the termination has one.
- The contributor documents say where a result is written:
  - `context/architecture/changes.md`, A command, owns the rule: `run` ends by writing its result through `output.rs`,
    which alone decides how a failed write ends, and the three endings are listed there.
  - `context/architecture/structure.md` gains the `output.rs` row of its Modules table.
  - `context/standards/rust.md`, Output and environment access: the sentence on a command writing through the handles of
    its `Context` links to that rule in place of describing a direct write.
- Files touched: `crates/core/src/output.rs` (new), `crates/core/src/{lib,termination,parse}.rs`,
  `crates/core/src/commands/ws/index.rs`, `crates/core/src/commands/repo/remove.rs`,
  `crates/core/tests/closed_stdout.rs` (new), `context/architecture/{structure,changes}.md`,
  `context/standards/rust.md`. No manifest changes: `serde_json` is already a dependency of the library, and the support
  package already a dev-dependency.
- `crates/dev/src/architecture.rs` and `crates/dev/tests/architecture.rs` pin nothing this change alters. The library
  token scan reads comments too, so no comment of `output.rs` spells a forbidden token of
  [repository-layout.md](../spec/repository-layout.md), Library rules, such as the standard stream paths.

## Acceptance

- Unit tests of `output.rs`, each with a handle the test owns:
  - `line` writes the text and one newline and returns `Success`; `text` writes the text unchanged and returns
    `Success`;
  - `document` writes one line that ends in a newline and parses back to the value given, for a value holding a double
    quote and a backslash, and returns `Success`;
  - with a handle whose every write fails with `ErrorKind::BrokenPipe`, each of the three returns
    `Termination::BrokenPipe`;
  - with a handle whose every write fails with another kind, each of the three returns
    `Termination::Internal(InternalError::Unexpected { .. })` whose diagnostic holds the error's text;
  - `document` makes exactly one `write` call with the whole line on a handle that accepts everything.
- Unit tests of `termination.rs`: `Termination::BrokenPipe.exit_code()` is 1 and its `diagnostic()` is `None`.
- `crates/core/tests/closed_stdout.rs`, component tests through `run`, with a stdout handle whose every write fails with
  `ErrorKind::BrokenPipe` and an in-memory stderr. Each asserts the termination is `Termination::BrokenPipe`, its exit
  code is 1, its diagnostic is `None`, and the stderr buffer is empty:
  - `--help`; `ws --help`; `ws index --help`; `--version`;
  - `ws index feature-x` and `ws index feature-x --json` in a `WorkspaceBuilder` workspace, whose tree is unchanged
    afterwards;
  - `repo remove project/web` and `repo remove project/web --json` in a workspace that declares the repository; the
    configuration file is the edited text afterwards, since the edit precedes the write of the result.
- The same file asserts that a bare root and a bare group, with that stdout handle, still end as
  `Termination::BareGroup` with exit code 2: nothing is written to stdout on that path.
- `grep -rnE 'InternalError|Backtrace|writeln!|write!\(|to_writer' crates/core/src/commands` prints nothing.
- `grep -n 'fn written' crates/core/src/parse.rs` prints nothing.
- The existing unit and component tests pass unmodified, apart from the two `the_written_document_is_one_line…` unit
  tests of the command files, which are replaced by the `document` tests of `output.rs`.
- `cargo dev gate full` exits 0, and the eight recorded scenarios pass.

## Out of scope

- An executable test with a real closed pipe: the process runner of [test-harness.md](../spec/test-harness.md) captures
  both streams and cannot close one; the rest of this epic adds it with the process boundary's own work.
- Writers for a result that is more than one line or document: reporter lines and NDJSON events of
  [output-formats.md](../spec/output-formats.md); the rest of this epic.
- The panic hook, the unwind to the boundary, and writing the backtrace: results-and-exit-codes.md, Internal failures;
  the rest of this epic.
- The usage line in a usage diagnostic, and a broken required-argument invariant reported as a usage error under a
  hand-typed command path: the rest of this epic.
- What a command file holds beyond its result: `command-file-rule`.
- A write-failure variant of `ConfigError`: `epic:parallel-workspaces/workspace-configuration`.
