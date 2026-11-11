# Workspace component engineering requirements

For `epic:parallel-workspaces/component-runtime`.

## What already holds

`engineering:rust-product-clis` constrains the implementation language. `epic:platform/cli-foundation` establishes the
platform's Rust executable and development-only sample process boundary, not a completed Workspace plugin API or
installed-product resolver; `epic:platform/launcher-shim` owns the production handoff.

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

The Workspace component consumes the production platform handoff once defined and preserves native argument/path values,
caller cwd, standard streams, and signal/exit behavior without re-parsing arguments through a shell. Invocation metadata
affects help presentation, not component identity or version. Serves `product:component-help-version` and
`product:automation-boundary`.

## Machine-output discipline (engineering:machine-output-boundary)

Commands that offer structured output emit their documented result without color, progress, provider chatter, or debug
logs on that output channel. Public result categories and compatibility tests are fixed before release under
`decision:command-vocabulary`; malformed extension output cannot be passed off as a valid Workspace result. Serves
`product:automation-boundary`.

## Governed contributor guidance (engineering:workspace-development-context)

Repository-owned context documents Rust conventions, command registration, dependency direction, wiring, errors,
reporting, test tiers, and concrete verification commands under the suite's shared context governance. Implementation
and review discover the same guidance through the code repository's entry point. Serves `product:governed-context`.

## Production handoff agreement (engineering:production-handoff-agreement)

Production component-description and invocation metadata consume the contract settled under
`decision:platform/launcher-shim/component-discovery`, including its agreement with invocation-aware help; the
development sample does not define that transport. This remains a production integration prerequisite for
`product:component-help-version` and `product:automation-boundary`, not a blocker to direct local execution.
