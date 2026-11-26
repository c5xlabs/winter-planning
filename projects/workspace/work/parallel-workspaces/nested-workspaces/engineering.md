# Nested workspace engineering requirements

For `epic:parallel-workspaces/nested-workspaces`.

## Verified child boundary before every authority change (engineering:child-authority-confinement)

Every child call runs the chain and marker preflight, and no mutating child call starts until a status read in the same
process has verified its resolved root; `--force` skips neither. Serves `product:verified-child-workspace`.

Acceptance:

- A `--force` destroy into a child whose status reports another root runs no child command.

## Nested invocation contract (engineering:nested-invocation-contract)

Every call into a child workspace runs `winter` as `decision:parallel-workspaces/nested-workspaces/child-compatibility`
selects it, with `WINTER_NESTED_CHAIN` holding the resolved roots of the calling chain, outermost first and joined by
the platform path-list separator, and refuses before spawning when its own root or the target root is already on the
chain. The child's environment is the caller's without any `WINTER_*` variable except `WINTER_LOG_LEVEL` and
`WINTER_OTEL_*`, and without `VIRTUAL_ENV` or its `bin` entry on `PATH`; a nested project's bootstrap `cmd` gets that
scrubbed environment without the chain. Serves `product:verified-child-workspace` and
`product:nested-teardown-preservation`.

Acceptance:

- A child that resolves back to a root already on the chain refuses its own nested calls without spawning.
- A child process and a bootstrap `cmd` see no `WINTER_*` variable other than `WINTER_LOG_LEVEL` and `WINTER_OTEL_*`.

## Child process environment isolation (engineering:child-process-isolation)

Child invocation crosses only what `engineering:nested-invocation-contract` keeps, so enclosing runtime scope,
code-selection overrides, and toolchain environment never reach the child. Arguments are not reconstructed through a
shell. Serves `product:verified-child-workspace` and `product:nested-local-inheritance`.

## Delegation writes are all-validated (engineering:delegation-write-integrity)

Allocation capacity and merged child configuration are validated before any delegation write; concurrent writers and
interruption cannot lose child-owned settings or publish a successful partial delegation. Serves
`product:nested-runtime-allocation` and `product:nested-local-inheritance`.

## Loss audit completeness is load-bearing (engineering:nested-loss-evidence)

Recursive deletion uses complete evidence for all child stores being removed; a skipped standalone probe, a configured
but missing tracking ref, a failed history read, or an incompatible child result cannot count as no local work.
Disposable multi-level Git fixtures cover retained branches, stashes, detached work, failed child env teardown, and a
failed child workspace-service stop. Serves `product:nested-work-inspection` and `product:nested-teardown-preservation`.
