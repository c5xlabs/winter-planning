# Workspace decisions

## Command vocabulary (decision:command-vocabulary)

**Decider:** product owner and engineering

**Open.** Choose the Workspace command groups, verbs, machine-output compatibility policy, and exit-code categories
before freezing contracts; this blocks finalizing `epic:parallel-workspaces/component-runtime` and the command-facing
requirements of the other Workspace epics, not their capability investigation.

## Configuration transition (decision:configuration-transition)

**Decider:** product owner and engineering

**Open.** Choose which existing configuration and machine-state formats are read directly and which require an explicit
conversion, including the component pin shared with Platform; this blocks finalizing
`epic:parallel-workspaces/workspace-configuration` and `epic:rust-adoption/workspace-migration`.

## Provider platform support (decision:supported-systems)

**Decider:** product owner and engineering

**Open.** Choose supported service-provider combinations and external runtime prerequisites within the shared CLI
baseline governed by `engineering:cli-runtime-baseline`; this blocks provider coverage acceptance for
`epic:rust-adoption/workspace-release`, not portable local development. Minimum OS/libc versions are settled once by
`decision:platform/release-automation/release-build-baseline`, not separately for Workspace.

## Observability shutdown budget (decision:observability-shutdown)

**Decider:** engineering

**Open.** Choose and measure a finite exporter shutdown budget, including unavailable endpoints and DNS behavior; this
blocks acceptance of `engineering:observability-isolation` as failure isolation, not a command-performance target.

## Development component selection (decision:development-component-selection)

**Decider:** product owner and Platform and Workspace engineering

**Open.** Choose the owner and supported workflow for selecting a modified Workspace build without replacing the
installed release or changing a workspace pin; this blocks finalizing
`product:parallel-workspaces/component-runtime/local-component-evaluation`, not direct local development. The current
launcher-shim epic excludes source builds and overrides, so this plan cannot imply that it already supplies them.

## Initiatives toward parity (decision:parity-initiative)

The rewrite has four outcome-based investments: `initiative:parallel-workspaces`, `initiative:running-environments`,
`initiative:agent-ready-operation`, and `initiative:rust-adoption`. Rough parity is their shared
`milestone:rust-workspace-parity`, not an initiative containing every capability; epic scope and requirements remain
intact while the folders and references follow these ownership boundaries.

## Rough parity boundary (decision:rough-parity)

Rough parity preserves the current Workspace capabilities and their safety guarantees, not Python's internal structure,
every old command spelling, or its in-process plugin API. The capability inventory and explicit redesign decisions bound
the rewrite; a capability is not silently dropped because its implementation does not translate directly.

## Rust replacement without Python delegation (decision:rust-replacement)

The replacement is a Rust `winter-workspace` component in the `winter-workspace` code repository, independently usable
and invoked through the platform's `winter workspace` group. It does not forward unfinished commands to the Python CLI
or require Python for first-party Workspace behavior; user-owned external programs remain the user's runtime choice.

## Requirements before contracts (decision:requirements-first)

This plan establishes capability initiatives, epics, product requirements, and engineering constraints before specs or
implementation slices. Areas that need a different design retain their observable promise and an owned open decision
instead of prematurely selecting a schema, Rust library, or plugin ABI.
