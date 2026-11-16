# Component runtime decisions

## Parsing of the trace export headers (decision:parallel-workspaces/component-runtime/trace-header-parsing)

**Decider:** product owner

Every item of `WINTER_OTEL_EXPORTER_OTLP_HEADERS` is read by one rule: its name and its value are percent-decoded and
trimmed, and the item is sent unless its decoded name is no HTTP token. The reference tool's helper reads an item in two
stages (`opentelemetry/util/re.py:52-93`): it decodes only an item in its strict form, and skips a value outside the set
its liberal form allows. It therefore sends a value that holds a raw space undecoded, skips an item whose value holds
`"`, `;`, `\`, a control byte, or a byte outside ASCII, and sends a name that is a token only before it is decoded.
Reproducing the two stages was rejected because the OpenTelemetry specification has header values percent-decoded and
the helper's exceptions are a library's leniency, so the three differences are a registered deviation.

## SIGINT ignored at start (decision:parallel-workspaces/component-runtime/interrupt-ignored-at-start)

**Decider:** product owner

When the binary starts with SIGINT's disposition already set to ignore — inherited from whatever started it: a
background job of a non-interactive shell, a `nohup`-style launcher, `trap '' INT` — it leaves SIGINT ignored and
installs no handler, as Unix convention has it and as the reference tool behaves: its language runtime leaves an ignored
SIGINT ignored, and no source of the reference tool installs a handler at start. Always installing the handler was
rejected because a stray SIGINT would then end, with exit 1, a run its launcher asked to shield.

## A `--` before a command name (decision:parallel-workspaces/component-runtime/double-dash-before-command)

**Decider:** product owner

A `--` standing before a command or group name is a usage error, exit 2: `-- ws index NAME` and `ws -- index NAME` are
refused, and a `--` after the leaf command (`ws index -- NAME`) ends option parsing as before. The reference tool only
ends option parsing at a `--` (`click/parser.py:329-330`) and a group reads the tokens left after it as the command
(`click/core.py:1534`), so it runs the command in both forms; the argument parser's own refusal was taken because nobody
writes `--` before a command name, and it is a registered deviation.

## Width of the termination (decision:parallel-workspaces/component-runtime/termination-payloads)

**Decider:** engineering

A `Termination` variant that carries an error type holds it in a `Box`, so the enum stays at most four words wide and is
returned by value as the error of every fallible function, however large its error types grow. Permitting a large error
type instead, by a lint attribute or a raised lint threshold, was rejected because the repository runs Clippy with its
defaults and keeps lint attributes to a closed list; boxing only the variants that are too large today was rejected
because every later growth of an error type would then change each site that builds or reads its ending.

## Trust roots of a trace export over TLS (decision:parallel-workspaces/component-runtime/trace-export-trust-roots)

**Decider:** product owner

An `https` trace endpoint is verified against the operating system's trust store, through the exporter's
`reqwest-rustls` feature: the OpenTelemetry crates are used as they ship, the binary bundles no roots, and a collector
whose certificate chains to a private authority installed on the machine is reachable over `https`. The reference tool
builds its exporter with no certificate argument and hides the generic exporter variables from it
(`core/internal/otel_command_tracer.py:43-58,177-179`), and its lock file has the exporter send through an HTTP library
that carries no root bundle (`tools/winter-cli/uv.lock`); that library is not in the reference tree and is believed to
verify against the language runtime's default trust locations, the system certificate store among them. The two agree
wherever those locations are the system's store, and a difference between them is a registered deviation.

Three other ways were rejected. An HTTP client built in the binary with bundled public roots and handed to the exporter
adds three direct dependencies and more code to the binary. Pinning the exporter at 0.31, which still has a feature with
bundled roots, starts the binary two releases behind. An exporter encoded by hand for `http` alone, with no dependency,
does not serve `https`.

Two consequences are accepted with it. `tokio` enters `Cargo.lock`, because the exporter's HTTP client depends on it
unconditionally and runs a runtime on a helper thread of its own, though the binary starts none. The build needs a C
compiler, because the TLS stack's cryptography (`aws-lc-sys`) compiles bundled C in its build script.

Four more are accepted with it, each read in the releases the crates resolve to. `Cargo.lock` grows by 161 packages, 27
of them with a build script and 12 procedural macros. The HTTP client reads the trust store when the exporter is built,
for an `http` endpoint as for an `https` one, and on a machine whose store yields no certificate tracing is unavailable
over both. `SSL_CERT_FILE` and `SSL_CERT_DIR`, which the certificate loader reads, name the store in place of the
system's and are inputs of the binary. A test that runs the real exporter needs a store: it names one inside its own
tree through `SSL_CERT_FILE`, and should that not hold, reading the host's store is accepted as an exception to the rule
that a test touches nothing outside its tree. Against these four, the exporter encoded by hand for `http` alone was
rejected again, and so was parking the export, which would leave the binary without its command span.

## Trace export transport (decision:parallel-workspaces/component-runtime/trace-export-transport)

**Decider:** product owner

The binary exports its command span through the standard OpenTelemetry Rust crates — the API, the SDK, and the OTLP
exporter over HTTP with protobuf — and serves `http` and `https` endpoints alike, as the reference tool does through its
SDK and HTTP library (`core/internal/otel_command_tracer.py:166-170`). An encoder and an HTTP client written in the
binary were rejected because they could serve `https` only by adding a TLS dependency of their own or by registering a
deviation.

## Arguments that are not UTF-8 (decision:parallel-workspaces/component-runtime/non-utf8-arguments)

**Decider:** product owner

A command-line argument that is not valid UTF-8 is a usage error, exit 2, wherever it stands on the command line and
whatever it would have meant: one rule for every argument position, after `--help`, `--version`, and `--` included. The
reference tool passes such bytes through to the command, which then succeeds or fails on them by its own rules; the
refusal is a registered deviation.

## Help or version beside a fault (decision:parallel-workspaces/component-runtime/help-beside-fault)

**Decider:** product owner

The tokens of a node are read left to right and the first of `--help`, `--version`, or a fault decides the ending:
`--version --bogus` prints the version line and exits 0, and `--bogus --version` is a usage error, exit 2. The reference
tool reads every token of the node before it answers, so the fault always wins there (`click/parser.py:294-310`); the
argument parser's own order was taken because no caller depends on the fault winning, and it is a registered deviation.

## Exit code of a provider ended by a signal (decision:parallel-workspaces/component-runtime/provider-signal-exit)

**Decider:** product owner

A service command whose provider entrypoint was ended by a signal exits with 128 plus the signal number, as a shell
reports it. The reference tool hands the negative code to its exit call unchanged (`modules/service/handler.py:68-73`)
and so exits with 256 minus the signal number, an accident of its runtime; the shell convention is a registered
deviation, and `epic:running-environments/service-orchestration` builds the mapping.

## Trace export for an interrupted run (decision:parallel-workspaces/component-runtime/interrupt-trace-export)

**Decider:** product owner

A run that SIGINT aborts outside the service paths exits at once and exports no span. The reference tool exports through
its single exit point (`cli.py:451-455`); a signal handler that exits cannot run an exporter soundly, so the missing
span is a registered deviation, and a run that ends through a returned termination, exit 130 included, still exports.

## Values of the command span's error type (decision:parallel-workspaces/component-runtime/span-error-type)

**Decider:** product owner

The command span's `error.type` attribute carries the binary's own name for the ending, from the closed vocabulary the
tracing spec lists. The reference tool sends its exception class names, such as `NoSuchOption`, `Abort`, and
`ConfigError` (`cli.py:150-162`), which name a language runtime the binary does not have; the change of values is a
registered deviation toward trace consumers.

## How the trace exporter is assembled (decision:parallel-workspaces/component-runtime/trace-exporter)

**Decider:** engineering

The OpenTelemetry crates are dependencies of the binary package alone, with a blocking HTTP client, a TLS stack that
needs no system library, and the SDK's unbatched processor, so the one span is exported once on the thread that ends it
and the binary starts no asynchronous runtime of its own; with no endpoint configured none of it is constructed. The
client runs a runtime on a helper thread it keeps to itself, and the TLS stack's cryptography is compiled from bundled C
when the binary is built. The batching processor was rejected because one span per process gains nothing from a
background worker, and a dependency of the library was rejected because `engineering:library-output-boundary` keeps
sockets out of it.

## Total bound on the trace export (decision:parallel-workspaces/component-runtime/trace-export-total-bound)

**Decider:** engineering

The binary holds the export cap itself: the span is ended and exported, and the provider shut down, on a thread the
tracer starts for it, and the ending waits for that thread for at most the cap, 100 ms from the start of the wait. On
expiry it abandons the thread without joining it and reports a failed export, and the thread ends with the process. The
exporter's own timeout stays at the cap as an inner bound, and is not the bound: in the HTTP client the crates ship it
starts afresh with every read of the response's body, and it does not cover the join of the client's helper thread
behind a name lookup that hangs. An HTTP client of the binary's own with one deadline was rejected because it is a
direct dependency of the binary, which using the crates as they ship rules out
(`decision:parallel-workspaces/component-runtime/trace-export-trust-roots`); accepting the bound of each read was
rejected because a collector that drips its response would then hold the command's exit without limit, which breaks the
cap.

## Repeated options (decision:parallel-workspaces/component-runtime/repeated-options)

**Decider:** product owner

A flag or a single-value option given more than once is a usage error, exit 2; only an option the command surface spec
marks repeatable may repeat. The reference tool accepts the repetition and lets the last value win
(`click/parser.py:165-178`), which hides a mistyped command line, so the refusal is a registered deviation.

## One parse pass (decision:parallel-workspaces/component-runtime/single-parse-pass)

**Decider:** engineering

The command line is parsed once, and the runtime derives from the resolved command path whether an invocation ended at
the root or inside a command. The reference tool parses the root, configures logging, and then parses the command
(`click/core.py:1866-1874`); two passes were rejected because the parser writes no log record, so the only observable
effects of the order are which endings open a command span and configure logging, and those follow from the path.

## Log records without a logging switch (decision:parallel-workspaces/component-runtime/unswitched-log-records)

**Decider:** engineering

With neither `-v` nor a valid `WINTER_LOG_LEVEL`, a record at WARNING or above is written to stderr as its message
alone, and lower records are dropped. The reference tool attaches no handler in that case (`cli.py:270-271`) and its
language runtime's handler of last resort writes exactly those records, which is what its users see; writing nothing was
rejected because the 61 warning sites would fall silent.

## Log level names (decision:parallel-workspaces/component-runtime/log-level-names)

**Decider:** engineering

`WINTER_LOG_LEVEL` accepts `DEBUG`, `INFO`, `WARNING`, `WARN`, `ERROR`, `CRITICAL`, `FATAL`, and `NOTSET`, in any case,
and ignores every other value silently; `NOTSET` switches the record format and leaves the threshold at warning, which
is what the reference tool's loggers do with it (`cli.py:288-292`). These are the level names the reference tool's
lookup resolves (`cli.py:277-285`); the five canonical names alone were rejected because `WARN` and `FATAL` are
spellings a caller may already set.

## Confirmation without a terminal (decision:parallel-workspaces/component-runtime/piped-confirmation)

**Decider:** engineering

A confirmation prompt reads its answer from stdin whether or not stdin is a terminal, as the reference tool does
(`click/termui.py`, `confirm`; `modules/workspace/handlers/destroy_handler.py:58-61`). Refusing without a terminal was
rejected because a script that pipes `y` would stop working; end of input still aborts before any mutation.

## Signals other than the interrupt (decision:parallel-workspaces/component-runtime/signal-disposition)

**Decider:** engineering

The binary handles SIGINT and leaves SIGTERM, SIGHUP, and SIGQUIT at their default disposition, as the reference tool
does: no source of it installs a handler (`core/internal/local_subprocess_runner.py:25-47` defers the interrupt only). A
handler that cleans up managed children on SIGTERM is a change of function for
`epic:running-environments/service-orchestration` to propose.

## A stdout that cannot be written (decision:parallel-workspaces/component-runtime/unwritable-stdout)

**Decider:** engineering

A write the reader's closed pipe refuses ends with exit 1 and nothing written, as the reference tool ends it
(`click/core.py:1425-1431`); any other failed write or flush is an internal failure with its diagnostic, exit 1; and a
process started with no stdout runs to its own ending, as the reference tool does when its stream is absent
(`click/utils.py:267-276`). One silent rule for every failed write was rejected because a full disk is a failure the
user must see.

## Fixture helper guarantees (decision:parallel-workspaces/component-runtime/fixture-helper-guarantees)

**Decider:** program lead, under the owner's constraint that tests never touch anything outside their tree; the owner
for where the guarantee ends

The git fixture helpers themselves refuse, before any effect and with symbolic links resolved, every path that does not
lie in the tree, refuse the tree's root as a repository, and name SHA-1 objects and the files ref backend on every
repository they create. A resolved check held only by the scenario fixture builder would leave component tests a weaker
boundary, and formats left to git's defaults would make the installed git's version part of every recorded object id.

The guarantee ends at a fixture built from honest inputs, which has no effect outside its tree, on the tree's top-level
entries, or on the network, and at the escapes the test harness contract lists, which the helpers refuse. They are no
sandbox: the ways a test can deliberately build a way out have no end to list, so a route the contract does not list is
outside the guarantee, and the standing rule that a test never touches anything outside its tree binds that test's
author.

## Git discovery stops at the isolated tree (decision:parallel-workspaces/component-runtime/git-discovery-containment)

**Decider:** program lead, under the owner's constraint that tests never touch a real workspace or repository

A tree is refused when its parent lies inside a git repository, and the pinned environment also sets
`GIT_CEILING_DIRECTORIES` so git's upward search for a repository stops at the tree. Either alone leaves a path open:
the refusal sees only the directories that exist when the tree is created, and the ceiling binds only git calls made
under the pinned environment.

## Gate scope (decision:parallel-workspaces/component-runtime/gate-scope)

**Decider:** product owner

The gates are Cargo only: the format check, Clippy with warnings denied, and the tests, with the recorded scenarios
added in the full gate. No Markdown or workflow lint step joins either gate, because those tools are installed outside
Cargo and the gates run on nothing but the pinned toolchain.

## Test harness provenance (decision:parallel-workspaces/component-runtime/harness-provenance)

**Decider:** product owner

The test harness is written fresh in this repository from the platform harness's pattern — an isolated tree, a pinned
environment, a bounded process runner — and no platform source is copied or adapted. The platform repository carries no
licence and this repository is FSL-1.1-ALv2, so the pattern is taken and the text is not.

## Command registration by file location (decision:parallel-workspaces/component-runtime/command-registration)

**Decider:** engineering

A command is one file under the library's command directory, built with the argument parser's builder interface; a build
script derives each command path from its file path and generates the module declarations and the registry, so adding a
command edits no tracked file. A hand-kept or committed registry makes every parallel command change collide in one
file, link-time registration hides a missing command until run time, and the parser's derive interface needs one shared
enum.

## Four packages, one library (decision:parallel-workspaces/component-runtime/crate-layout)

**Decider:** engineering

The repository is one Cargo workspace of four packages: the binary, one library holding every command and domain module,
test support, and the development driver. A package per domain was rejected for now because every new package edits the
shared workspace and binary manifests, and module boundaries inside one library are enough until a domain needs its own
dependency set.

## Gates run plain Cargo through one driver (decision:parallel-workspaces/component-runtime/gate-driver)

**Decider:** engineering

Each gate is one subcommand of a Rust development binary reached through a Cargo alias, because an alias cannot chain
commands and a shell script would be a second language to lint. Recorded scenarios are one test target of the binary
package selected by a Cargo feature that changes no compiled code, which keeps them out of the slice gate and gives them
the built binary's path from Cargo.

## One CI workflow that runs the full gate (decision:parallel-workspaces/component-runtime/ci-workflow)

**Decider:** engineering

The reference tool's CI workflow is deleted with the reference tool, and the repository carries one workflow whose only
build step is the full gate command, so CI and a local run cannot drift. Keeping no workflow was rejected because
`engineering:reproducible-build` requires CI to run the same checks; the workflow is not exercised until the branch is
pushed.

## Root documents kept through the wipe (decision:parallel-workspaces/component-runtime/retained-root-documents)

**Decider:** engineering

`LICENSE` and `PRINCIPLES.md` stay on the Rust branch unchanged: the licence covers the repository whatever its
language, and the principles state why the product exists without naming the reference tool, its commands, or its files.
Every other root document is rewritten because each one instructs the reader to install or run the reference tool.

## Git failure rendering (decision:parallel-workspaces/component-runtime/git-failure-rendering)

**Decider:** product owner

A failed git call's text reaches stderr once, in the repository error that reports it; it appears additionally as a log
record only when logging is turned up with `-v` or `WINTER_LOG_LEVEL`. The reference tool logs the failure at error
level before raising it (`modules/workspace/internal/repo_error_factory.py:103`), and with no handler attached that
record reaches stderr as a bare line, so its users see the text twice; printing it once is a registered deviation.

## Internal failure rendering (decision:parallel-workspaces/component-runtime/internal-failure-rendering)

**Decider:** product owner

An unexpected failure, a panic included, ends with one error line on stderr and exit 1, where the reference tool prints
a traceback and exits 1. A backtrace follows only when `RUST_BACKTRACE` is set, which is a local development aid.

## Program name (decision:parallel-workspaces/component-runtime/program-name)

**Decider:** product owner

The binary is `winter-workspace` and names itself so in usage, help, and `--version`, whatever path invoked it. The
`config.lock` header comment and trace span names keep the reference tool's literal `winter`, because committed files
and trace consumers carry them.

## Launcher-only surface is absent (decision:parallel-workspaces/component-runtime/launcher-only-surface)

**Decider:** product owner

Commands, options, and files that exist only in the reference tool's bash launcher, such as `init` and `--winter=PATH`,
are not part of the binary; the command surface spec lists each one as deliberately absent. `WINTER_INVOCATION_CWD` and
`WINTER_LAUNCH_TIME`, which the launcher sets and the reference tool itself reads, stay inputs the binary honours with
the reference fallback when unset, though nothing in this program sets them.

## Help rendering (decision:parallel-workspaces/component-runtime/help-rendering)

**Decider:** engineering

Help goes to stdout with exit 0 for `--help`, and to stderr with exit 2 for a bare group or a root given options and no
command. It lists the commands and options the command surface spec names and omits hidden ones; its wording, order, and
layout are the argument parser's own.

## Error rendering (decision:parallel-workspaces/component-runtime/error-rendering)

**Decider:** engineering

Errors are plain text on stderr in every mode, each naming the subject its error kind carries. `--json` adds no error
envelope; consumers decide success from the exit code.

## Exit-code overloads (decision:parallel-workspaces/component-runtime/exit-code-overloads)

**Decider:** engineering

`ws status` exits 1 for a dirty or drifted worktree and 2 when the command itself fails. A service command returns the
exit code of a provider that ran, with three exceptions (`core/internal/local_subprocess_runner.py:267-280`,
`modules/service/handler.py:105-109`): 126 when a provider entrypoint could not be executed and 1 when
`service up --wait` times out after the provider succeeded, both as the reference tool has them, and 128 plus the signal
number for a provider ended by a signal, under `decision:parallel-workspaces/component-runtime/provider-signal-exit`. An
interrupt exits 130 only on the service paths and 1 everywhere else.

## JSON encoding (decision:parallel-workspaces/component-runtime/json-encoding)

**Decider:** engineering

JSON output is any valid JSON encoding of the reference tool's document: the same keys, values, nesting, and event
kinds, with one document per line where the shape is a stream. Whitespace, key order within an object, character
escaping, and the wording of a human message carried as a string value are not contract; such a value keeps its place
and type.

## JSON-mode purity (decision:parallel-workspaces/component-runtime/json-mode-purity)

**Decider:** engineering

Where the reference tool writes a plain-text line to stdout under `--json`, the binary writes a plain-text line there
too, in its own wording. The output formats spec lists every such command.

## Colour policy (decision:parallel-workspaces/component-runtime/color-policy)

**Decider:** engineering

Colour follows the reference tool's terminal detection, and `NO_COLOR` is ignored.
