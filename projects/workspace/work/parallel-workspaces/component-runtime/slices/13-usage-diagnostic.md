---
name: usage-diagnostic
depends-on:
  - component-test-scaffold
code:
  - winter-workspace/crates/core/src/termination.rs::UsageError
  - winter-workspace/crates/core/src/termination.rs::Termination
  - winter-workspace/crates/core/src/parse.rs
  - winter-workspace/crates/core/src/process.rs::run
  - winter-workspace/crates/core/src/context.rs::Context
  - winter-workspace/crates/core/src/commands/ws/index.rs::params
  - winter-workspace/crates/core/src/commands/repo/remove.rs::params
---

# The parser answers, the run dispatches, and the usage diagnostic carries its usage line

Builds the seam of [runtime.md](../spec/runtime.md), The parse and the run, the usage diagnostic of
[results-and-exit-codes.md](../spec/results-and-exit-codes.md), Usage diagnostic and Error rendering, and `Invoked`,
`Usage`, and `arguments::required` of [runtime.md](../spec/runtime.md), What a command receives and Reading a parsed
argument, under `engineering:typed-outcomes`, `product:component-help-version`, and `product:automation-boundary`
("invalid operands and unknown commands report usage without starting the requested mutation"). A command file no longer
types its own command path, and a parsed value that is missing against the command's own parser is an internal failure,
not a usage error. `Termination` holds each error it carries in a `Box`, under
`decision:parallel-workspaces/component-runtime/termination-payloads`, so the functions this change adds return it as
the `Err` of a `Result`. Every later change of this epic builds on the seam this one lands, so the changes after it are
built in their numbered order, one at a time, except `stdout-fault-tests` and `interrupt`, which may be built beside
each other.

## Change

- `parse.rs`: `parse` is the pure function the spec gives. It answers `Parsed` with `Stage` and `Outcome`; it takes no
  `Context` and no handle, writes nothing, and calls no command. `Stage::Command` holds `invoked` and nothing else yet:
  `RootArgs` arrives with `root-options`. The one pass fills `Usage` from the node the fault belongs to, and the command
  path follows the spec's deciding-token rule. Its private constructor of the internal failure is gone.
- `process.rs`: `run` is `run_with` over the registry, with step 1 and step 3 of the spec's order: it writes
  `Outcome::Help` and `Outcome::Version` through `output.rs` on the handle the `Invocation` carries, returns
  `Outcome::Ended`, and for `Outcome::Dispatch` builds the one `Context`, now with `invoked`, and calls the command.
  `run_in_memory` exists under the test configuration.
- `context.rs`: `Invoked` exists, with the attribute the command contract carries; `Context` gains `invoked`.
- `termination.rs`: `Usage` exists; `UsageError::Parse` gains `usage`. `diagnostic()` renders every error ending as the
  Error rendering table gives it, the usage diagnostic in its three parts; `ParseFault::UnexpectedArgument` renders its
  values as the spec says. `InternalError::unexpected` exists as [runtime.md](../spec/runtime.md), The process boundary,
  gives it, and `parse.rs`, `arguments.rs`, and `process.rs` build the variant through it. `UsageError::Rule` and
  `UsageRule` are not built: no command of the tree refuses a usage rule yet.
- `termination.rs`: the four variants of `Termination` that carry an error type hold it in a `Box`, as
  [results-and-exit-codes.md](../spec/results-and-exit-codes.md), Domain, gives them: `Usage(Box<UsageError>)`,
  `Command(Box<CommandError>)`, `Config(Box<ConfigError>)`, and `Internal(Box<InternalError>)`. The error types
  themselves hold no new box. Every site of the library that builds one of the four endings boxes the error there —
  `parse.rs`, `process.rs`, `arguments.rs`, `output.rs`, and the two command files — and every test that reads the error
  of an ending reads it through the box. `InternalError::unexpected` answers an `InternalError`, as the spec gives it,
  and its caller boxes the answer.
- No source of the repository carries a lint attribute for the size of an error (`clippy::result_large_err`,
  `clippy::large_enum_variant`), and the repository gains no `clippy.toml` and no lint table.
- `crates/core/src/arguments.rs` exists with `required` as the spec gives it.
- `commands/ws/index.rs`, `commands/repo/remove.rs`: `params` returns `Result<Params, Termination>` and reads each
  required value through `arguments::required`. Neither file holds a string literal of its own command path, and neither
  builds a usage error.
- In-crate tests: the tests of `parse.rs` that asserted on what a run wrote or returned assert on the `Parsed` they are
  answered, or go through `process::run_in_memory` with entries of their own; no test of the library builds a `Context`
  or an `Invocation` outside `process.rs`.
- The scaffold's `InProcess` needs no new input: `Invoked` is derived by the parser.
- Contributor documents, each a sentence this change makes false or a rule it adds:
  - `context/architecture/structure.md`, Modules: the `parse.rs` row ("the argument parser, built from the registry, and
    the dispatch to a command") and the `process.rs` row state the seam; the `arguments.rs` row is added.
  - `context/architecture/changes.md`: Writing the result, "the parser's help and version path writes through the same
    module", names `process.rs` as the caller; A command says how `params` reads a value; An error says that a new
    `Termination` variant holds its error type in a `Box`, with a link to the rule in `rust.md`; the sentence on the
    unit test in `parse.rs` that checks the registry stays true and is left.
  - `context/standards/rust.md`: the sentence "Three items of the library carry `#[allow(dead_code…)]` permanently"
    names four, `Invoked` among them; Errors says a broken parser invariant is an internal failure built through the
    constructor `InternalError::unexpected`. Errors also gains the rule on the width of `Termination`, stated there
    once: a variant that carries an error type holds it in a `Box`; the fields a variant holds inline are scalars or one
    `String`; a fallible function of the library returns `Result<_, Termination>` by value; no lint attribute and no
    lint setting is the answer to an error that is too large; and the unit test of `termination.rs` named below checks
    the width. Lint attributes stays as it is: Clippy runs with its defaults.
- Files touched: `crates/core/src/{termination,parse,context,process,output,lib}.rs`, `crates/core/src/arguments.rs`
  (new), `crates/core/src/commands/ws/index.rs`, `crates/core/src/commands/repo/remove.rs`,
  `crates/core/tests/{root,ws_index,repo_remove}.rs`, `crates/cli/tests/root.rs`,
  `context/architecture/{structure,changes}.md`, `context/standards/rust.md`, and the scenario directories below. No
  manifest changes. `crates/core/src/output.rs` changes only where it builds and where its tests read the internal
  failure, which is now boxed: its functions already take a handle, and its private constructor stays until
  `process-boundary`. `crates/core/tests/support/in_process.rs` is not touched: `Ended.termination` already carries the
  usage. `crates/cli/src/main.rs` is not touched: it reads an ending only through `exit_code()` and `diagnostic()`.
- `crates/dev/src/architecture.rs`: the command-file rule's deny-list and the forbidden tokens are unchanged.
  `arguments.rs` spells none of the forbidden tokens, and a command file reaches the internal failure only through
  `arguments::required`, so it still names neither `InternalError` nor `Backtrace`. `crates/dev/tests/architecture.rs`
  pins nothing this change alters.
- Scenarios recorded with `cargo dev record`, compared on exit code alone since each writes only human text:
  `root/unknown-command`, `root/unknown-option`, `ws/unknown-command`, `ws-index/missing-name`,
  `ws-index/extra-argument`. Component tests hold the diagnostic's shape, which no scenario compares. `root` is the
  directory of an invocation whose command path is empty (`engineering.md`, Open questions).

## Acceptance

- Unit tests of `parse.rs`, each asserting on the `Parsed` of a command line: the stage and the command path of every
  row of the spec's table that needs no root option (`ws index NAME`, `ws index --bogus`, `ws index`, `ws --help`,
  `ws nope`, `ws`, `--version`, `--help`, `--bogus`, `nope`, `--bogus ws index`, and no arguments); the outcome is
  `Dispatch` for the first, `Help` for a `--help`, `Version` for `--version`, and `Ended` otherwise.
- Unit tests of `termination.rs`: for each `ParseFault` variant, `diagnostic()` has the three parts in order, the usage
  line is the `Usage` given, and the help command names the node's path; every other error ending starts with the
  error-line marker.
- A unit test of `termination.rs`: `std::mem::size_of::<Termination>()` is at most four times
  `std::mem::size_of::<usize>()`.
- `grep -rnE 'result_large_err|large_enum_variant' crates` prints nothing, and the repository holds no `clippy.toml`.
- Component tests, through the scaffold:
  - an unknown command at the root, an unknown command under `ws`, an unknown option of `ws index`, a missing `NAME`,
    and an extra argument each end as `Termination::Usage` with exit code 2, and the `usage` names the node the fault
    belongs to: the root, `ws`, and `ws index` respectively;
  - two extra arguments render both, each quoted;
  - `--help` and `--version` write to stdout and end as `Success`, as before.
- A unit test of `arguments.rs`: `required` on an id the matches do not hold returns `Termination::Internal`, exit code
  1.
- `grep -rnE '"(ws index|repo remove)"' crates/core/src/commands` prints nothing.
- `grep -n 'fn internal' crates/core/src/parse.rs` prints nothing.
- `grep -rnE 'Context \{|Invocation \{' crates/core/src --include='*.rs'` prints lines of `process.rs` and of the
  definitions in `context.rs` only.
- `sed '/#\[cfg(test)\]/,$d' crates/core/src/parse.rs | grep -nE 'cx\.|\bContext\b'` prints only the lines that import
  `Context` and name it in the type of the registration's function: the check reads the code above the file's
  `#[cfg(test)]` line. The test module below it keeps entries of its own, whose run functions take a `Context`.
- `crates/cli/tests/root.rs`: the built binary given an unknown command exits 2 with empty stdout and a stderr of at
  least three lines.
- `workspace:manual-registration` passes, since the change touches `crates/core/src/parse.rs`
  (`context/verification/matrix.md`).
- The five scenarios pass, and `cargo dev gate full` exits 0.

## Out of scope

- An argument that is not UTF-8, a repeated option, the root options, a bare root given options, and help or version
  beside a fault: `root-options`.
- `UsageError::Rule`: the first command that refuses a usage rule, in its own epic.
- The panic hook, and the constructor's use in `output.rs`: `process-boundary`.
