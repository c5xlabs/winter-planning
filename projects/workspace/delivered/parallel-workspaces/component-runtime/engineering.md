# Workspace component engineering requirements

For `epic:parallel-workspaces/component-runtime`.

## What already holds

`engineering:rust-product-clis` constrains the implementation language. `epic:platform/cli-foundation` establishes the
platform's Rust executable and development-only sample process boundary, not a completed Workspace plugin API or
installed-product resolver. Workspace is invoked directly and reads no launcher handoff, under
`decision:rust-replacement`.

## Rust-only code branch (engineering:rust-only-branch)

The branch that carries the Rust binary holds no source, test, packaging, skill, or manual of the reference tool, and no
root document that describes it. The reference tool stays readable and runnable only outside that branch, under
`decision:rust-replacement`.

Acceptance:

- The tracked file set of the branch contains no path under `tools/`, `skills/`, or the reference tool's `context/`
  tree, and no Python source, Python project file, or Python CI job.
- The root `README.md`, `AGENTS.md`, `CLAUDE.md`, `.gitignore`, `dprint.json`, and `.rumdl.toml` name no reference-tool
  command, path, or install step.

## Reproducible local development (engineering:reproducible-build)

The Workspace repository pins its Rust toolchain and dependency resolution, documents build prerequisites, and exposes
format, lint, check, focused-test, and aggregate verification commands. The local loop needs neither Python nor an
installed Winter executable and does not replace commands on PATH. Serves `epic:parallel-workspaces/component-runtime`.

Acceptance:

- Fresh-checkout builds use the checked-in locks without rewriting them; build output is untracked.
- After dependency bootstrap, local checks can run without credentials, network access, or live providers.
- CI runs the same required checks, including rustfmt and Clippy, with compiler and lint warnings treated as failures.
- Check mode does not rewrite source; required checks cover production code, test code, and every enabled target and
  feature. Repository-owned Markdown and the CI workflow file are outside the gates, under
  `decision:parallel-workspaces/component-runtime/gate-scope`.

## Two gates, one command each (engineering:verification-gates)

The repository has a slice gate and a full gate. Each is one command run from the repository root, runs every one of its
steps even after a step fails, names the failed steps, and exits 0 only when all passed. The slice gate is rustfmt in
check mode, Clippy with warnings denied, and the test suite without recorded scenarios; the full gate is the slice gate
plus every recorded scenario of `epic:rust-adoption/parity-validation`, under
`decision:parallel-workspaces/component-runtime/gate-scope`. Neither gate executes Python, reads the network, or
rewrites a tracked file. Serves `epic:parallel-workspaces/component-runtime`.

Acceptance:

- A formatting fault, a Clippy warning, a compiler warning, and a failing test each make the slice gate exit non-zero
  and name the step that failed.
- A failing recorded scenario makes the full gate exit non-zero and leaves the slice gate at 0.
- Running either gate on a clean checkout leaves `git status` clean.
- Every step of either gate starts the Cargo executable that started the gate and no other program.

## Testable production boundaries (engineering:testable-boundaries)

Workspace has separately runnable unit, component, and executable/service tests with shared test support, and automatic
checks reject forbidden dependency edges and production dependencies on test code. Argument parsing, command behavior,
domain logic, external effects, and output handling have declared responsibilities. Serves
`epic:parallel-workspaces/component-runtime`.

Acceptance:

- Domain and command behavior can be tested without launching a process; executable tests exercise the built production
  implementation rather than an independent behavior model.
- Executable tests use an explicit built path and isolated cwd, HOME, configuration, and environment snapshots.
- Test subprocesses have finite deadlines and are terminated and reaped on failure; real user workspaces and installed
  executables are not test fixtures.

## Library never owns a stream (engineering:library-output-boundary)

The library crate writes only to the output handles its caller passes in and returns a typed termination; it never opens
the process's standard streams, never exits the process, and never reads the process environment or working directory
except through values the caller passes. The binary crate alone binds the real streams, environment, and working
directory, writes the final diagnostic, and maps the termination to an exit code. Inside the library, files are reached
only through the file port, which applies `engineering:domain-io-boundaries`. Serves `product:automation-boundary` and
applies `engineering:composition-root`.

Acceptance:

- A check over the library's sources fails when one names the process's standard output, standard error, process exit,
  or a print macro, or names the filesystem module outside the file port's own source.
- A component test runs a command in process with in-memory output handles and reads its result and termination without
  starting a process.

## Additive command registration (engineering:additive-registration)

Adding a command adds files and edits none: the command's parser, its dispatch entry, its tests, and its recorded
scenarios are each a new file that the build or the test runner discovers by location. Two changes that each add a
different command merge without touching a common tracked file. Serves `epic:parallel-workspaces/component-runtime`.

Acceptance:

- A change that adds one command file under the command directory makes the command appear in help and dispatch with no
  other tracked file changed.
- A file in the command directory that does not satisfy the registration contract fails the build with a message naming
  the file.
- An alias that names no sibling command fails the test suite, naming the alias file.

## Native invocation fidelity (engineering:handoff-fidelity)

The binary is invoked directly and takes its arguments, caller cwd, standard streams, and signals from the operating
system as given, without re-parsing arguments through a shell and without reading handoff metadata. The name in usage
lines and the version line is fixed and never derived from the invoking path. Serves `product:component-help-version`
and `product:automation-boundary`.

## Machine-output discipline (engineering:machine-output-boundary)

Commands that offer structured output write their result to stdout without color, progress, provider chatter, or debug
logs; the only non-JSON text on that channel is the set the output formats spec lists as JSON-mode departures, under
`decision:parallel-workspaces/component-runtime/json-mode-purity`. Result shapes are fixed under
`decision:command-vocabulary`; malformed extension output cannot be passed off as a valid Workspace result. Serves
`product:automation-boundary`.

## Governed contributor guidance (engineering:workspace-development-context)

Repository-owned context documents Rust conventions, command registration, dependency direction, wiring, errors,
reporting, test tiers, and concrete verification commands under the suite's shared context governance. Implementation
and review discover the same guidance through the code repository's entry point. The context covers building and
verifying the binary; it is not a manual of the binary's commands. Serves `product:governed-context` and
`product:local-component-evaluation`.

Acceptance:

- Repository-owned agent/runtime guidance gives the actual build command and resulting executable path for local
  `winter-workspace` development, with examples for a feature-environment worktree.
- Examples distinguish the source/build directory from the command's target cwd, preserve the intended target workspace,
  and explain that a locally built executable has the same mutation authority as an installed component.
- Guidance tells agents to rebuild after source changes, invoke the built sub-executable directly, and avoid replacing
  installed commands or writing mutable build output into the released-version cache.
- Guidance identifies retained `--winter=PATH` as a Python-only implementation override, not a way to select a Rust
  component or change the target workspace.
- The repository's agent entry point routes to this guidance; essential invocation instructions do not live only in the
  planning repository or a human-only development guide.
- Every document under the repository's context tree is reachable from the root `AGENTS.md` by following links.
- The verifiability matrix gives each verification method a scoped id and one command, or for a manual method its setup
  and success condition; each listed command exists and exits 0 on a clean checkout.
- No context document describes a command's user-facing usage beyond what a contributor needs to run the binary from the
  checkout.

## Production handoff agreement (engineering:production-handoff-agreement)

Production component-description and invocation metadata consume the contract settled under
`decision:platform/launcher-shim/component-discovery`, including its agreement with invocation-aware help; the
development sample does not define that transport. This remains a production integration prerequisite for
`product:component-help-version` and `product:automation-boundary`, not a blocker to direct local execution.

## Open questions

- The command span's contract is the tracing spec, under `engineering:observability-isolation`. It covers the command
  span and its one export. Spans of single operations, the `winter.env` attribute, trace context handed to child
  processes, and the dashboard's session spans have no contract yet; engineering of the epic that owns each command, and
  of the first epic that starts a child process, writes it there.
- The port that starts a child process has three duties the specs of this epic already place on it and no slice of this
  epic builds: it builds the child's environment from the snapshot and never inherits the process environment, from
  which the binary removes the generic exporter variables while tracing is on; it leaves `WINTER_LAUNCH_TIME` out; and
  it defaults `GIT_SSH_COMMAND`. Engineering of the first epic whose command starts a child process — by the order of
  the backlog, `epic:parallel-workspaces/repository-catalog` or the first after it that calls git — specifies the port.
- Whether `PATH` and `HOME` are rows of the process-input table: the binary of this epic starts no child and reads
  neither. The same epic adds a row for each variable the binary itself consults to find or start a program, and leaves
  out what only the child reads.
- The runtime spec gives the confirmation prompt, the writer for a result of several pieces, and the notice writer as
  contracts with no caller in this epic. Engineering of the epic whose command first prompts, streams events, or writes
  a notice builds each one with that command; the same holds for `Termination::Aborted`, `Termination::Interrupted`, and
  `UsageError::Rule`, which the results spec defines and the first command that returns each one adds.
- Recorded scenarios of an invocation that ends at the root sit in a `root/` directory, which the scenario format's rule
  "the command path joined by `-`" does not name for an empty path. Engineering of
  `epic:rust-adoption/parity-validation` states the directory name for an empty path in the scenario format; until then
  `root` is used, and the runner accepts any directory name.
- The gate driver has four known gaps that no acceptance line above reaches. Engineering closes each with the next
  change to the driver, and none is planned in this epic:
  - a SIGHUP to the driver ends it and leaves the running step alive;
  - the driver's interrupt path, and the flag that makes `cargo dev fmt` rewrite files, have no test;
  - that a step runs with its stdin closed is the driver's behaviour and is not stated in the gates spec;
  - the driver's tests build their directories with a helper of their own where the isolated tree of the test harness
    spec would serve.
