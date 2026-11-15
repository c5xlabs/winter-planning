# Runtime

What surrounds a command from process start to exit: the process boundary, the seam between the parser and the run, the
values a command receives, the root options, logging, the owners of everything written to either stream, prompts,
interrupts, and the test seams that hold them. The command tree and the parser's answers are
[command-surface.md](./command-surface.md); the endings and their exit codes are
[results-and-exit-codes.md](./results-and-exit-codes.md); streams and JSON shapes are
[output-formats.md](./output-formats.md); the command span is [tracing.md](./tracing.md); packages and files are
[repository-layout.md](./repository-layout.md); the three single-result writers are
[command-registration.md](./command-registration.md), Writing a result. Reference locations are paths under
`tools/winter-cli/src/winter_cli/` of the `winter-workspace` repository at `b57b020`; `click/` is the pinned Click 8.3.2
the reference tool runs on.

## System

```text
# engineering:library-output-boundary, engineering:handoff-fidelity
crates/cli/src/main.rs            the process boundary: snapshot, streams, panic hook, the call of run, the final
                                  flush, the diagnostic, the backtrace, the exit code
crates/cli/src/interrupt.rs       the SIGINT handler and the one static Interrupts value
crates/cli/src/trace.rs           the binary's tracer and its one export; tracing.md
crates/core/src/process.rs        run, Invocation, Identity, ProcessEnv, Interrupts; what follows the parse: help and
                                  version written, logging resolved, the one Context built, the command called
crates/core/src/context.rs        Context, Invoked, RootOptions
crates/core/src/parse.rs          the parser and what it answers: Parsed, the root's arguments, the usage of the node
                                  at fault. It writes nothing and calls no command.
crates/core/src/arguments.rs      reading a parsed argument that the parser guarantees
crates/core/src/logging.rs        Level, Log, and the resolution of -v and WINTER_LOG_LEVEL
crates/core/src/diagnostics.rs    the one owner of what the library writes to the stderr handle
crates/core/src/output.rs         the one owner of what reaches the stdout handle
crates/core/src/trace.rs          the pure part of tracing and the tracer port; tracing.md
crates/core/src/confirm.rs        the one owner of a confirmation prompt
crates/core/tests/support/        the in-process scaffold of the component tests
```

`arguments.rs`, `logging.rs`, `diagnostics.rs`, and the three tracing files are built by this epic. `confirm.rs`,
`output::Results`, and `diagnostics::notice` are specified here and built by the change that adds their first caller,
because the gate denies code that nothing calls ([repository-layout.md](./repository-layout.md), What a command
receives).

## Domain

### What the binary hands over

```rust
// crates/core/src/process.rs — the fields of repository-layout.md, Domain, with three changes
pub struct Invocation<'a> {
    pub args: Vec<std::ffi::OsString>,
    pub cwd: std::io::Result<std::path::PathBuf>,   // the error of std::env::current_dir() is kept, not flattened
    pub env: ProcessEnv,
    pub identity: Identity,
    pub stdin: &'a mut dyn std::io::BufRead,
    pub stdout: &'a mut dyn std::io::Write,
    pub stderr: &'a mut dyn std::io::Write,
    pub stdout_is_terminal: bool,
    pub stderr_is_terminal: bool,
    pub interrupts: &'a Interrupts,
    pub tracer: &'a dyn CommandTracer,              // tracing.md, The library's part; added with the tracer
}

// One value per process. The binary's signal handler and the library share it.
pub struct Interrupts { /* two atomics: deferred, pending */ }
impl Interrupts {
    pub const fn new() -> Self;          // not deferred, not pending
    pub fn is_deferred(&self) -> bool;   // read by the handler
    pub fn note(&self);                  // called by the handler for every interrupt, first: sets pending
    pub fn pending(&self) -> bool;       // read by a command that deferred, after its release; never cleared
    pub fn defer(&self);                 // a command takes the interrupt over: service paths only
    pub fn release(&self);               // back to the default
}
```

```text
unreadable working directory
  `cwd` is an Err when the process has no readable working directory (it was removed, or a parent denies search).
  --version, every --help, a bare root or group, and every parser usage error do not read it and end as they always do.
  A command that is dispatched ends as Termination::Internal(InternalError::Unexpected) with that error as source,
  exit 1, before `run` of the command file is called. The reference tool fails on the first read of the working
  directory with an unhandled error, exit 1 (config/internal/cwd_workspace_locator.py:26-33).
```

### The parse and the run

One statement of who does what between the argument vector and a command. Every other spec and every change of this epic
takes it from here.

```rust
// crates/core/src/parse.rs — the parser. Pure: it takes no handle and no Context, writes nothing, and calls no
// command. decision:parallel-workspaces/component-runtime/single-parse-pass
pub(crate) fn parse(entries: &[Entry], identity: &Identity, args: &[std::ffi::OsString]) -> Parsed;

pub(crate) struct Parsed {
    pub stage: Stage,
    pub outcome: Outcome,
}

// How far the command line got before the token that decided the outcome.
pub(crate) enum Stage {
    Root,                                          // no command name resolved
    Command { invoked: Invoked, root: RootArgs },  // at least one did
}

pub(crate) enum Outcome {
    Help(String),              // the rendered help of a node, asked for with --help: for stdout, exit 0
    Version(String),           // the version line, without its newline: for stdout, exit 0
    Ended(Termination),        // Termination::Usage or Termination::BareGroup; or, when the parser cannot be built
                               // from the registry, Termination::Internal through InternalError::unexpected, at
                               // Stage::Root; nothing else
    Dispatch {                 // a command was selected and its arguments parsed
        run: fn(&clap::ArgMatches, &mut Context<'_>) -> Termination,   // the registration's function
        matches: clap::ArgMatches,                                     // of the selected node
    },
}

// The root's own options as typed, before the environment is consulted.
#[derive(Debug, Default, PartialEq, Eq)]
pub(crate) struct RootArgs {
    pub verbose: bool,
    pub service_orchestrator: Option<String>,
    pub source_override: Option<String>,
}

// The one place the root's arguments and the environment become values. Pure.
pub(crate) fn root_values(args: &RootArgs, env: &ProcessEnv) -> (RootOptions, Log);
```

```text
the command path and the stage
  The deciding token is the first argument that is not UTF-8, when the line holds one: it is refused wherever it
  stands, also after a --help or a --version (command-surface.md, Parsing rules). Otherwise it is the first of
  --help, --version, or a token at fault. The command path is the command names the parser resolved among the tokens
  before it; a root option's value is no command name. When no token decides before the end of the line — a
  dispatch, a bare group, a missing required argument — the path is every name resolved.
    ws index NAME      [ws, index]        ws index --bogus   [ws, index]        ws index           [ws, index]
    ws --help          [ws]               ws nope            [ws]               ws                 [ws]
    --version   --help   --bogus   nope   --bogus ws index   -v -v ws index NAME   (no arguments)      []
  An empty path is Stage::Root. Any other is Stage::Command, whose Invoked holds the path and the usage of its last
  node, and whose RootArgs holds the root options given before the first command name. The parser has accepted
  those tokens by then whether or not the rest parsed; when the parse failed and left no matches, `parse` reads
  them again against the root's option definitions alone.
```

```rust
// crates/core/src/process.rs
pub fn run(invocation: Invocation<'_>) -> Termination;      // run_with over the generated registry
pub(crate) fn run_with(entries: &[Entry], invocation: Invocation<'_>) -> Termination;
```

```text
run_with, in order                                  # the endings: results-and-exit-codes.md, Flows
  1. parsed = parse(entries, &identity, &args)
  2. Stage::Command only:
       (options, log) = root_values(&root, &env)
       the runtime's Debug record (Diagnostics on the stderr handle), written to Invocation.stderr
       tracer.command_started(&invoked.path, ..)      tracing.md; with the tracer
     Stage::Root resolves no logging, writes no record, and announces nothing.
  3. by outcome
       Help(text)       output::text(Invocation.stdout, &text); its answer is the ending
       Version(line)    output::line(Invocation.stdout, &line); its answer is the ending
       Ended(ending)    the ending; nothing is written here: the boundary writes its diagnostic
       Dispatch         Invocation.cwd is read: an Err ends as What the binary hands over gives. Otherwise the one
                        Context of the invocation is built from the Invocation's parts, the Invoked, the RootOptions,
                        and the Log, and `run` is called with the matches and that Context. Its answer is the ending.
```

Help and the version line are written before any `Context` exists, by `run_with`, on the handle the `Invocation`
carries; `output.rs` takes a handle, not a `Context`. `process.rs` holds the one statement that builds a `Context`, and
the one call of a command. An in-crate unit test never builds a `Context`: a test of the parser asserts on the `Parsed`
it is answered, and a test that needs a command of its own run — the tests of `parse.rs` that register entries of their
own, and those of `output.rs`, `arguments.rs`, and `diagnostics.rs` that need an ending — goes through one helper:

```rust
// crates/core/src/process.rs, test configuration only — the one in-crate site that builds an Invocation
#[cfg(test)]
pub(crate) fn run_in_memory(entries: &[Entry], args: &[&str]) -> (Termination, String, String);   // stdout, stderr
// cwd "/", an empty environment, empty stdin, neither stream a terminal, a fresh Interrupts, NoTrace
```

### What a command receives

```rust
// crates/core/src/context.rs — the fields of repository-layout.md, What a command receives, and four more
pub(crate) struct Context<'a> {
    // cwd, env, identity, stdin, out, err, out_is_terminal, err_is_terminal, files, file_writer: as built
    pub invoked: &'a Invoked,
    pub root: &'a RootOptions,
    pub log: Log,
    pub interrupts: &'a Interrupts,
}

// The node being run, as the user typed it. For an alias the path holds the alias name.
pub(crate) struct Invoked {
    pub path: Vec<String>,     // ["service", "start"]
    pub usage: Usage,
}

// crates/core/src/termination.rs — what a usage diagnostic shows of the command at fault. Public through the
// facade: UsageError carries it, and Invoked above holds one.
#[derive(Debug, Clone, PartialEq, Eq)]
pub struct Usage {
    pub line: String,          // the usage line of the node, as its --help prints it
    pub help_command: String,  // "<program> <path…> --help"
}

// crates/core/src/context.rs — the root option values a command can read. Verbosity is not here: it reaches
// logging and nothing else (reference: cli_context.py:34-51, cli.py:341-346).
#[derive(Debug, Default, PartialEq, Eq)]
pub(crate) struct RootOptions {
    pub service_orchestrator: Option<std::ffi::OsString>,   // the effective override, resolved against the variable
    pub source_override: Option<String>,
}
```

```text
# reference: cli.py:336,345
service_orchestrator   --service-orchestrator VALUE when VALUE is not empty
                       else WINTER_SERVICE_ORCHESTRATOR when set and not empty, its bytes as given
                       else None: the provider comes from configuration
source_override        --source-override VALUE as given; None when the option is absent
an option value is text: an argument that is not UTF-8 never reaches here (command-surface.md, Parsing rules)
the Log of root_values is Log::resolve over the root's verbose flag and WINTER_LOG_LEVEL (Logging)
```

A command file reads `cx.root` and `cx.invoked`; it never parses a root option itself, and it never types its own
command path. `Invoked` and `RootOptions` are part of the command contract and carry the attribute `Context` carries
([repository-layout.md](./repository-layout.md), What a command receives): commands that read each field arrive over
many changes.

### Logging

```rust
// crates/core/src/logging.rs
#[derive(Debug, Clone, Copy, PartialEq, Eq, PartialOrd, Ord)]
pub(crate) enum Level { Debug, Info, Warning, Error, Critical }

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub(crate) struct Log {
    pub threshold: Level,   // records below it are dropped
    pub switched: bool,     // whether -v or a valid WINTER_LOG_LEVEL switched the record format
}
impl Log {
    pub(crate) const DEFAULT: Log = Log { threshold: Level::Warning, switched: false };
    // decision:parallel-workspaces/component-runtime/log-level-names
    pub(crate) fn resolve(verbose: bool, variable: Option<&std::ffi::OsStr>) -> Log;
}
```

```text
# reference: cli.py:264-293,332 — tests/test_cli.py::TestConfigureLogging
resolve
  verbose                              Log { Debug, switched }           -v wins over the variable
  variable names a level, any case     Log { that level, switched }
     DEBUG  INFO  WARNING  WARN  ERROR  CRITICAL  FATAL
     WARN is WARNING, FATAL is CRITICAL
  variable is NOTSET, any case         Log { Warning, switched }
     the reference tool sets level 0 on its two named loggers and on the handler (cli.py:288-292); a logger at
     level 0 takes the level of the root logger, which nothing in the reference tool configures and which is
     WARNING: records at WARNING and above, in the switched format
  variable empty, unknown, or not UTF-8   Log::DEFAULT; nothing is written about it
  neither                              Log::DEFAULT
when
  logging is resolved once, at Stage::Command (The parse and the run), before the command's `run`. An invocation
  that ends at the root resolves nothing and writes no record (reference: cli.py:324-332, click/core.py:1866-1872).
```

## Interfaces

### The process boundary

```rust
// crates/cli/src/main.rs — whole responsibility. engineering:library-output-boundary
fn main() -> std::process::ExitCode {
    // 1. interrupt::install(&INTERRUPTS); install the panic hook for every thread
    // 2. snapshot argv without argv[0], std::env::current_dir() as a Result, ProcessEnv::from_vars(vars_os());
    //    ask stdout and stderr whether each is a terminal; build the tracer from trace::settings (tracing.md)
    // 3. code = conclude(
    //        || { lock stdin, stdout, stderr; run(Invocation { .. }) },     // the locks end with the closure
    //        || std::io::stdout().flush(),
    //        &mut std::io::stderr(),
    //        |error_type, stderr| tracer.finish(error_type, stderr),
    //    )
    // 4. ExitCode::from(code)
}

// The part of the boundary a unit test drives with in-memory handles and closures of its own.
fn conclude(
    run: impl FnOnce() -> Termination,
    flush: impl FnOnce() -> std::io::Result<()>,      // the final flush of stdout
    stderr: &mut dyn std::io::Write,
    finish: impl FnOnce(Option<&'static str>, &mut dyn std::io::Write),   // the span's end and export; tracing.md
) -> u8;
```

The four arguments share no borrow: `std::io::stdout()` and `std::io::stderr()` each answer a handle to the process's
one stream, so the closure passed as `run` takes its own locked handles and releases them when it returns, `flush` takes
a handle of its own afterwards, and `stderr` is a third. `conclude` calls its closures one after another and never two
at once. The `finish` parameter, step 5 below, and `Invocation.tracer` are added by the change that builds the tracer;
before it `conclude` has the first three parameters.

```text
conclude, in order
  1. arm the panic slot of the calling thread; call `run` inside std::panic::catch_unwind. A panic becomes
     Termination::Internal(InternalError::Panic { message, backtrace }) from what the hook stored for this thread.
  2. call `flush`. When it fails and the termination's exit code is 0, the termination becomes
     Termination::from_write_error(error); a termination that already failed stands.
  3. write termination.diagnostic(), when Some, to `stderr`, with a newline added when the text lacks one.
  4. for Termination::Internal whose backtrace has status Captured: write the backtrace to `stderr`.
  5. call `finish` with termination.error_type() and `stderr`, inside catch_unwind; a panic there is dropped.
  6. answer termination.exit_code().
  A failed write to `stderr` in step 3 or 4 is ignored: the exit code stands and nothing further is tried.
panic hook                # decision:parallel-workspaces/component-runtime/internal-failure-rendering
  replaces the default hook; for a thread it covers it writes nothing and stores the panic's message (the payload
  when it is a &str or a String, otherwise a fixed text) with its location, and
  std::backtrace::Backtrace::capture(), in a slot keyed by the panicking thread's id. `conclude` empties the slot of
  its own thread. Exit code 101 never occurs.
  which threads it covers
    installed by `main`: every thread of the process. A panic on a second thread is stored under that thread's id,
    and the thread that joins it takes the stored value by that id and turns the join error into
    InternalError::Panic.
    installed by a unit test of `conclude`: only a thread `conclude` armed. A panic on any other thread goes to the
    hook that was installed before, so a failing assertion of another test is reported as always, and two tests
    that panic at once each read their own message.
  the backtrace of step 4 is the one the termination carries: a unit test hands `conclude` a `run` that answers a
  Termination::Internal built with Backtrace::force_capture(), and one built with Backtrace::disabled(), and does
  not depend on RUST_BACKTRACE of the test process.
no stdout at all
  with file descriptor 1 closed at start (`>&-`), the standard library accepts and drops every write. The command
  runs to the ending it would have had and writes nothing to stdout (reference: click/utils.py:267-276).
```

```rust
// crates/core/src/termination.rs — the one classifier of a failed stdout write or flush; output.rs calls it for a
// write the library makes, and the boundary for the final flush
impl Termination {
    pub fn from_write_error(error: std::io::Error) -> Termination;
    // ErrorKind::BrokenPipe -> Termination::BrokenPipe; any other -> Termination::Internal(Unexpected { .. })
}
impl InternalError {
    // The one constructor of an unexpected failure: it captures the backtrace. parse.rs, output.rs, arguments.rs,
    // and process.rs build the variant through it and hold no constructor of their own.
    pub(crate) fn unexpected(source: impl Into<Box<dyn std::error::Error + Send + Sync>>) -> InternalError;
}
```

### Interrupts and signals

```text
# crates/cli/src/interrupt.rs — reference: cli.py:434-436, core/internal/local_subprocess_runner.py:25-47
SIGINT    the disposition the process started with is read before anything else runs, by a sigaction call that
          passes no new action and asks for the previous one
            ignore                     -> nothing is installed and SIGINT stays ignored for the whole run
                                          (decision:parallel-workspaces/component-runtime/interrupt-ignored-at-start)
            anything else              -> the handler is installed with a second sigaction call, with SA_RESTART
                                          and an empty mask
          the query comes first, in place of an installation that is undone after: until the second call the
          disposition is the inherited one, so no interrupt is ever handled against the launcher's wish
          under SA_RESTART a call the system can restart resumes when the handler returns into it: a read, a
          write, a wait. poll, select, epoll_wait, and nanosleep are never restarted and still return interrupted,
          so a command that defers handles that error from those
          the handler, in order
            INTERRUPTS.note()          always, and first
            INTERRUPTS.is_deferred()   -> the handler returns
            otherwise                  -> write the aborted notice to file descriptor 2 with write(2), then _exit(1)
          a handler that runs with no state stored notes nothing and ends the process the same way: the notice,
          then _exit(1)
          the note comes before the test: the signal may land on a second thread, and with the test first a command
          could release and read pending() between the test and the note, and the interrupt would be lost
          a command that deferred reads pending() after its release(), never before, and when it is true ends as
          Termination::Interrupted, exit 130: a pending interrupt always ends the invocation, and pending is not
          cleared within one. The reference tool delivers a pending interrupt again when its deferral ends
          (core/internal/local_subprocess_runner.py:42-48); here that duty is the command's.
          the handler calls only async-signal-safe functions: atomic loads and stores, write, _exit
          it flushes nothing and exports no span
          (decision:parallel-workspaces/component-runtime/interrupt-trace-export)
          nothing but `service logs`, `service status`, and readiness polling ever defers
SIGTERM, SIGHUP, SIGQUIT
          no handler: the default disposition ends the process; no notice, no flush, no cleanup of children
          (the reference tool installs none either: no `signal.` call in its sources)
SIGPIPE   ignored, as the Rust runtime sets it at start: a write to a closed pipe fails with BrokenPipe and ends as
          Termination::BrokenPipe (reference: click/core.py:1425-1431)
children  a child process started in the terminal's foreground process group receives the terminal's SIGINT itself
```

The undeferred handler ends the process itself and returns no `Termination`. `Termination::Interrupted` and
`Termination::Aborted` are each added by the command that first returns one
([results-and-exit-codes.md](./results-and-exit-codes.md), Domain); this epic builds neither.

`unsafe` in the binary is in two places, each carrying the comment the Library rules ask for. In `interrupt.rs`: the two
zeroed `sigaction` structures, the two `sigaction` calls and the `sigemptyset` between them, the handler's one
dereference of the address of its state, and its `write` and `_exit`; and what its tests use, the `raise` and the
`signal` call by which each fixture sets the disposition of SIGINT it starts with. In `trace.rs`: the one block that
removes the generic exporter variables from the process environment ([tracing.md](./tracing.md), The binary's part). The
handler's state is a static `AtomicPtr<Interrupts>`: null until the installer stores the address of the
`&'static Interrupts` it is given, which it does before it installs the handler, and never changed after. The binary
package's third-party dependencies are `libc` and the tracing crates of [repository-layout.md](./repository-layout.md),
The Cargo workspace.

```text
testing the handler                     # test-harness.md, tier table
  a unit test of interrupt.rs starts a child test of the same test binary through the support package's `invoke`.
  The child sets the disposition of SIGINT it is to start with, default or ignore, installs the handler on an
  Interrupts of its own, writes a line that says it is ready, and raises SIGINT on itself with raise(3). No test
  sends a signal to another process, so no test needs a child's process id or a readiness protocol: the parent
  reads the child's exit, stdout, and stderr once it has ended.
    not deferred    the child's exit is 1, its stderr holds the notice, and nothing follows the ready line
    deferred        the handler returns; the child reads pending() as true and exits 0
    ignored         the child sets SIGINT to ignore itself, then calls the installer; after the raise it runs on,
                    reads pending() as false, and exits 0, and its stderr holds no notice
```

### Reading a parsed argument

```rust
// crates/core/src/arguments.rs — the parser guarantees a required argument and a defaulted option; a command file
// reads them through this function and never builds a usage error for their absence.
pub(crate) fn required<'m, T>(matches: &'m clap::ArgMatches, id: &str) -> Result<&'m T, Termination>
where T: Clone + Send + Sync + 'static;
// Err: Termination::Internal(InternalError::Unexpected { source: MissingArgument { id }, .. }) when the id is
// absent, unknown to the node, or of another type. Never a panic, never a usage error.
```

```rust
// a command file's parameter mapping, whole shape
fn params(matches: &clap::ArgMatches) -> Result<Params<'_>, Termination> {
    Ok(Params { name: arguments::required::<String>(matches, "name")?, json: matches.get_flag("json") })
}
fn run(matches: &clap::ArgMatches, cx: &mut Context<'_>) -> Termination {
    let params = match params(matches) { Ok(params) => params, Err(ended) => return ended };
    /* one call into a domain module; the mapping of its outcome */
}
```

### Diagnostics on the stderr handle

```rust
// crates/core/src/diagnostics.rs — the one owner of what the library writes to the stderr handle. A command, a
// domain module, and the runtime write there only through it. Each call hands one whole line, newline included, to
// the handle in one write_all. A failed write is dropped: stderr never changes how an invocation ends.
pub(crate) struct Record<'r> {
    pub level: Level,
    pub origin: &'r str,              // the module that reports, e.g. "parse"
    pub message: &'r str,
    pub only_when_switched: bool,     // decision:parallel-workspaces/component-runtime/git-failure-rendering
}
pub(crate) fn record(err: &mut dyn std::io::Write, log: Log, record: &Record<'_>);

// A line that is no log record: a warning, a reporter failure line, a retry notice, the notice that a service
// orchestrator override is in effect. Written whatever the threshold.
pub(crate) fn notice(err: &mut dyn std::io::Write, line: &str);
```

```text
record
  level below log.threshold                     dropped
  only_when_switched and not log.switched       dropped
  log.switched                                  a line carrying the level, the origin, and the message
  not switched (WARNING and above)              the message alone
the runtime's own record
  step 2 of run_with (The parse and the run): one Debug record, origin "parse", naming the command path. It is
  written at Stage::Command whatever the outcome — a dispatch, a command's --help, and a command-stage usage error
  alike — and it is the record that shows -v and WINTER_LOG_LEVEL=DEBUG working on a command that logs nothing else.
the two records about tracing: tracing.md, The library's part
```

### Writing more than one piece of a result

```rust
// crates/core/src/output.rs — beside line, text, and document (command-registration.md, Writing a result).
// For a result of several lines, an event stream, or events followed by a document. The first failed write is the
// ending of the invocation: the command returns the Err it is handed and writes nothing more to stdout.
pub(crate) struct Results<'o> { /* the stdout handle */ }
impl<'o> Results<'o> {
    pub(crate) fn new(out: &'o mut dyn std::io::Write) -> Self;
    pub(crate) fn line(&mut self, line: &str) -> Result<(), Termination>;                       // text, "\n"
    pub(crate) fn document(&mut self, document: &serde_json::Value) -> Result<(), Termination>; // one line of JSON
    // One NDJSON event: one line of JSON, then a flush of the handle, so a reader sees the event when it happens
    // (output-formats.md, JSON shapes, "framing").
    pub(crate) fn event(&mut self, event: &serde_json::Value) -> Result<(), Termination>;
}
// Err is what Termination::from_write_error gives for the failed write or flush.
```

### Confirmation

```rust
// crates/core/src/confirm.rs — product:automation-boundary. The one owner of a prompt.
// reference: click/termui.py (confirm); modules/workspace/handlers/destroy_handler.py:58-61
// Ok(()): the answer was yes. Err: how the invocation ends — the command returns it before any mutation.
pub(crate) fn confirm(
    out: &mut dyn std::io::Write,
    input: &mut dyn std::io::BufRead,
    question: &str,
) -> Result<(), Termination>;
```

```text
confirm
  writes the question and a marker that the default is no to `out`, without a newline, and flushes
  reads one line from `input`
    y, yes (any case, surrounding blanks ignored)     Ok(())
    n, no, or an empty line                           Err(Termination::Aborted { interrupted: false })
    end of input, or a read error                     Err(Termination::Aborted { interrupted: false })
    anything else                                     asks again
  asking again has no bound: the question is repeated for every line that is no answer, and the loop ends only with
  an answer or with the end of input, as the reference tool's does
  a failed write of the question                      Err(Termination::from_write_error(..))
no terminal check        # decision:parallel-workspaces/component-runtime/piped-confirmation
  the answer is read from whatever stdin is: a pipe that supplies `y` confirms
which commands prompt, when, and what skips a prompt: results-and-exit-codes.md, Interrupts and prompts
```

The winter process reads stdin for confirmation and for nothing else. A child process inherits stdin unless the port
that starts it says otherwise (reference: `core/internal/local_subprocess_runner.py:20-22,192-218`).

### Terminal, colour, width

```text
# decision:parallel-workspaces/component-runtime/color-policy
terminal detection   made once by the binary, per output stream, and carried as out_is_terminal / err_is_terminal
colour               a command styles its human result only when the stream it writes to is a terminal
                     help, the version line, usage diagnostics, and error diagnostics are never styled
width                help is rendered at one fixed width, the argument parser's own when it is built without its
                     terminal-size feature: 100 columns. The terminal size and COLUMNS are not read, so the same
                     command line prints the same help in a terminal of any width and into a pipe. The reference
                     tool wraps at the terminal's width, at most 80 columns; layout is free
                     (decision:parallel-workspaces/component-runtime/help-rendering).
variables            the runtime reads none of NO_COLOR, FORCE_COLOR, CLICOLOR, CLICOLOR_FORCE, TERM, COLUMNS, LINES,
                     LANG, LC_ALL, or any other LC_ variable; setting one changes nothing the binary writes
encoding             every byte the binary writes is UTF-8, whatever the locale
```

### Test seams

```rust
// crates/core/tests/support/in_process.rs — engineering:testable-boundaries. The one scaffold that runs the library
// in process. A component test file declares it with `#[path = "support/in_process.rs"] mod in_process;`.
// Cargo builds no test target from a subdirectory of tests/ that holds no main.rs.
pub struct InProcess { /* args, cwd, env pairs, stdin bytes, terminal flags, an Interrupts; defaults below */ }
impl InProcess {
    pub fn new(args: &[&str]) -> Self;      // cwd "/", empty environment, empty stdin, neither stream a terminal,
                                            // no interrupt pending
    pub fn os_args(args: Vec<std::ffi::OsString>) -> Self;     // the same, for an argument that is no text; with
                                                               // the refusal of such an argument
    pub fn cwd(self, cwd: &std::path::Path) -> Self;
    pub fn cwd_error(self, error: std::io::Error) -> Self;     // Invocation.cwd is this Err; with the cwd Result
    pub fn env(self, key: &str, value: &str) -> Self;
    pub fn stdin(self, bytes: &[u8]) -> Self;
    pub fn interrupt_pending(self) -> Self;                    // deferred and noted; with Interrupts
    pub fn run(self) -> Ended;                                  // in-memory stdout and stderr
    pub fn run_with_stdout(self, stdout: &mut dyn std::io::Write) -> Ended;   // a handle of the test's own
}
pub struct Ended {
    pub termination: Termination,
    pub stdout: String,
    pub stderr: String,
    pub announced: Option<Vec<String>>,     // the command path `run` gave the tracer port; with the tracer
}
```

Each constructor, setter, and field marked "with …" is added by the change that builds what it names. The scaffold holds
no writer type: a test that needs a stdout that fails keeps that writer in its own file and hands it to
`run_with_stdout`. The scaffold file opens with `#![allow(dead_code, reason = "…")]`: each test target uses part of it.
It is the one lint attribute for unused code outside the command contract of repository-layout.md.

The scaffold is for the component tests, which are outside the library. A unit test inside the library uses
`run_in_memory` (The parse and the run). The process runner that gives the built binary a stdout it cannot write,
`invoke_with_stdout`, is [test-harness.md](./test-harness.md), Interfaces.

## Flows

The order of one invocation, with every ending, is [results-and-exit-codes.md](./results-and-exit-codes.md), Flows.

## Open questions

- `Interrupts::defer` leaves open what a deferred command does with its children on SIGTERM; engineering answers it when
  `epic:running-environments/service-orchestration` is refined.
