# Workspace engineering requirements

For Winter Workspace.

## What already holds

- `engineering:rust-product-clis` and `engineering:release-versions` bind Workspace without a separate language or
  release policy. `product:governed-context`, `decision:shared-context-governance`, and `decision:context-dogfooding`
  govern its contributor context; Rust conventions follow that governance, not copied Python tool requirements.
- `engineering:cli-runtime-baseline` supplies shared CLI targets and OS/libc compatibility; Workspace owns only its
  provider-specific support and external runtime prerequisites under `decision:supported-systems`.
- `decision:rust-replacement`, `decision:rough-parity`, and `decision:requirements-first` establish the replacement
  scope without freezing contracts. `epic:parallel-workspaces/component-runtime` owns reproducible development, test
  facilities, dependency checks, invocation fidelity, machine-output handling, and contributor guidance.
- `decision:platform/launcher-shim/component-boundary` keeps Workspace behind a separate executable. Platform owns
  discovery and version selection; `decision:platform/launcher-shim/component-discovery` and
  `decision:platform/launcher-shim/version-selection` remain open. The foundation's sample is not a production protocol.
- `decision:platform/version-cache` owns the released-component cache. `decision:shared-release-capability` and
  `product:homebrew-installation` supply the shared release and installation baseline;
  `epic:rust-adoption/workspace-release` owns Workspace integration, not ordinary development prerequisites.

Source alignment and unresolved ownership are recorded in
[research findings](./research/strategy-alignment/findings.md).

## Explicit application wiring (engineering:composition-root)

An explicit composition root binds collaborators and supplies them through constructors or explicit call arguments;
Workspace uses neither a DI container nor a service locator. Services do not construct hidden collaborators, and runtime
creation dependencies are explicit factories rather than access to the application graph.

## Domain and external-system boundaries (engineering:domain-io-boundaries)

Domain behavior consumes domain values and narrow module configuration, not the whole application configuration schema;
configuration loading and whole-workspace reconciliation are explicit exceptions. External I/O stays behind declared
adapter seams, with read-only authority separated from mutations whose ordering and safety belong to domain services.

## Typed operation results (engineering:typed-outcomes)

Domain results distinguish completion, refusal, and partial execution with typed outcomes and structured target state;
operational failures retain typed causes rather than becoming success booleans or message parsing. Handling and
rendering occur at the boundary that recovers or reports the failure, without repeated catch-log-propagate noise. Serves
`product:explain-results`.

## Winter-owned provider contracts (engineering:provider-contract)

Built-in and external providers implement Winter-owned operations with the same conformance obligations; callers cannot
depend on arbitrary backend flag passthrough. Provider-specific translation stays behind the adapter, not in domain
orchestration or a second public command contract.

## External code boundary (engineering:external-code-boundary)

Workspace never loads extension code into its process or delegates unfinished behavior to the legacy CLI; first-party
behavior, including built-in providers, has no Python runtime obligation under `decision:rust-replacement`. External
extensions run as separate programs behind a versioned contract; user-owned programs retain their own runtime choices.

## Explicit shell execution (engineering:process-execution)

Process execution uses native argument vectors without a shell by default; shell evaluation requires an explicit
operator-authored configuration entry and is never inferred from command text. That exception documents its visible
environment and substitution rules, rejects prohibited command-derived substitutions before execution, and does not
claim to prevent the shell from expanding values present in its environment.

## Concurrent authoritative state (engineering:concurrent-state-safety)

Conflicting authoritative updates are serialized or refused, and successful concurrent updates cannot silently overwrite
one another; independent scopes retain separate mutable state and resource ownership. Failed or interrupted writes leave
valid committed state or a diagnosed recoverable transition, not truncated authoritative files or an unreported partial
mutation. Serves `product:preserve-unrelated-work` and `product:explain-results`.

Acceptance:

- Competing same-scope operations do not lose an accepted update or report an inconsistent state as complete.
- Injected interruption and write failures preserve recoverable authoritative content; cross-system partial effects are
  recorded rather than presented as a transaction that never happened.
- Concurrent different-scope operations do not replace each other's state or claim each other's managed resources.

## Bounded, non-interfering observability (engineering:observability-isolation)

Observability preserves operation context across parallel work and nested invocations without leaking secrets, raw
arguments, configuration values, or command output into span attributes, or invocation trace context into managed
service launches. Exporter failure cannot change operation outcomes or wait indefinitely at exit; exporter shutdown is
bounded by the finite budget chosen in `decision:observability-shutdown` so failed telemetry cannot hang command exit.
