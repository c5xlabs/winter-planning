# Workspace component engineering requirements

For `epic:parallel-workspaces/component-runtime`.

## What already holds

`engineering:rust-product-clis` constrains the implementation language. `epic:platform/cli-foundation` establishes the
platform's Rust executable and development-only sample process boundary, not a completed Workspace plugin API or
installed-product resolver. Workspace is invoked directly and reads no launcher handoff, under
`decision:rust-replacement`.

## Reproducible local development (engineering:reproducible-build)

The Workspace repository pins its Rust toolchain and dependency resolution, documents build prerequisites, and exposes
format, lint, check, focused-test, and aggregate verification commands. The local loop needs neither Python nor an
installed Winter executable and does not replace commands on PATH. Serves `epic:parallel-workspaces/component-runtime`.

Acceptance:

- Fresh-checkout builds use the checked-in locks without rewriting them; build output is untracked.
- After dependency bootstrap, local checks can run without credentials, network access, or live providers.
- CI runs the same required checks, including rustfmt and Clippy, with compiler and lint warnings treated as failures.
- Check mode does not rewrite source; required checks cover production code, test code, enabled targets/features,
  repository-owned Markdown, and development/CI automation, with narrow documented exclusions.

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
and review discover the same guidance through the code repository's entry point. Serves `product:governed-context` and
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

## Production handoff agreement (engineering:production-handoff-agreement)

Production component-description and invocation metadata consume the contract settled under
`decision:platform/launcher-shim/component-discovery`, including its agreement with invocation-aware help; the
development sample does not define that transport. This remains a production integration prerequisite for
`product:component-help-version` and `product:automation-boundary`, not a blocker to direct local execution.
