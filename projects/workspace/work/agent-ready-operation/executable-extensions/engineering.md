# Executable contribution engineering requirements

For `epic:agent-ready-operation/executable-extensions`.

## What already holds

`engineering:external-code-boundary` excludes in-process extension imports and first-party Python delegation.
`engineering:process-execution` supplies argument-vector and explicit-shell rules; `engineering:observability-isolation`
constrains tracing and secret handling. Provider operations remain owned by
`epic:running-environments/service-orchestration`, not by a generic extension transport.

## Versioned external participation (engineering:external-participation)

Executable contributions cross a versioned subprocess boundary using serializable context and results, not host-language
objects, mutable domain references, or UI callbacks. Workspace integration and legacy compatibility cannot be finalized
before `decision:agent-ready-operation/executable-extensions/protocol-integration` resolves shared ownership and
contract dependencies; no production API shape is selected here. Serves `product:compatible-contributions` and
`product:inspect-contributions`.

## Execution limits and authority (engineering:invocation-authority)

Every executable contribution has explicit cwd, environment inheritance, stdin, interaction, deadline, output-limit, and
process-tree cleanup policies. Scope-specific managed variables cannot leak from a caller into a different scope, and
execution failure cannot contaminate the structured result channel. Serves `product:controlled-execution`.

Acceptance:

- Escaping or symlink-redirected executable paths are refused before launch according to the declared trust boundary.
- Timeout and cancellation terminate and reap the processes Workspace owns within the agreed finite cleanup budget;
  detached external resources are reported separately rather than claimed to be reaped.
- Invalid or oversized response records fail within the agreed limit without unbounded buffering or waiting for a hung
  child; continuous log streams do not acquire a finite total byte or duration limit by accident.
- Tests inject ambient feature-scope variables into workspace-scope invocations and verify the declared scope policy.

## Hook prerequisites and repeatability (engineering:hook-phase-accounting)

Hook dispatch records phase prerequisites, attempts, skips, and results independently from structural and provisioning
results. Automatic retries are prohibited until the hook's repeatability contract permits them, and strict teardown
cannot erase already-completed earlier effects from its report. Serves `product:lifecycle-contributions`.

## Isolated conformance effects (engineering:conformance-isolation)

Conformance execution defaults to non-mutating probes or isolated stand-ins; invoking real provider action words with a
synthetic name is not proof of isolation. Effectful checks require an isolated target, bounded subprocesses, and
authorization separate from ordinary inspection. Serves `product:safe-conformance`.
