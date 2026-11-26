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

This epic registers `--strict` on `ws destroy` and adds `Options.strict` to
[destroy.md](../../parallel-workspaces/environment-lifecycle/spec/destroy.md), Domain, as its System table assigns; its
`on_env_destroy` phase reads it, and the same change sets `Forwarded.strict` of
[teardown.md](../../parallel-workspaces/nested-workspaces/spec/teardown.md), Forwarded, from it, so each nested child's
`ws destroy` receives the option, which nested-workspaces leaves to the option's owner.

Acceptance:

- With `--strict`, a failed `on_env_destroy` hook ends the target with `StrictAbort` naming the hook phase and keeps the
  env's worktrees; without it, the failure is reported and the teardown continues.
- A `ws destroy --force --strict` of an env holding a nested child starts the child's `winter ws destroy <env>` with
  `--force` and `--strict`, and one without `--strict` starts it without.

## Isolated conformance effects (engineering:conformance-isolation)

Conformance execution defaults to non-mutating probes or isolated stand-ins; invoking real provider action words with a
synthetic name is not proof of isolation. Effectful checks require an isolated target, bounded subprocesses, and
authorization separate from ordinary inspection. Serves `product:safe-conformance`.

## Program calls in recorded scenarios (engineering:program-call-capture)

A recorded scenario whose command starts another program replaces that program with a recording stub, and replay
compares the calls the stub received with the reference tool's. The harness is built to the `[[stub]]` table and Program
calls of [scenario-format.md](../../../delivered/rust-adoption/parity-validation/spec/scenario-format.md) and the
`Calls` row of [record-and-replay.md](../../../delivered/rust-adoption/parity-validation/spec/record-and-replay.md),
under `decision:rust-adoption/parity-validation/program-capture-placement`, before the first command of this epic that
starts a program. Serves `product:controlled-execution`.

Acceptance:

- Where the invocation hands work to another program, the record also stores the argument vector, working directory, and
  contract environment variables each such program received.
- Where a command hands work to another program, replay fails on a different argument vector, working directory, or
  contract environment variable given to that program.
