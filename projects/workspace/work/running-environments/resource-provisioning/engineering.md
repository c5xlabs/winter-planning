# Resource provisioning engineering requirements

For `epic:running-environments/resource-provisioning`.

## What already holds

`engineering:external-code-boundary` and `engineering:process-execution` govern user commands and executable extensions.
`epic:running-environments/scope-environment` supplies runtime values, while
`epic:parallel-workspaces/environment-lifecycle` remains authoritative for environment identity and structural changes.
Stage execution does not depend on complete service orchestration; required-service checks and composed bring-up are
requirement-level integrations with it, not prerequisites for the independent preparation engine.

## Explicit execution plan (engineering:provision-plan-authority)

Selection, action composition, ordering, scope, and source attribution have one semantic definition shared by preview
and execution; validation never executes configured commands as a planning side effect. Skips and unresolved preflight
conditions are retained in the plan outcome. Serves `product:ordered-preparation`, `product:targeted-resource-actions`,
and `product:declared-cleanup-scope`.

## Recoverable aggregate outcomes (engineering:provision-failure-boundaries)

Execution preserves per-command, per-cwd, per-handler, and per-environment outcomes without treating external effects as
rolled back. Apply/destroy/reset short-circuit boundaries differ explicitly from best-effort clean; operational
exceptions cannot bypass the aggregate partial-result boundary. Serves `product:preparation-partial-results`.

## Separate service integration (engineering:provision-service-seam)

Required-service checking consumes service control and observation only for selected applicable handlers; it is not a
prerequisite for collecting or executing independent stages. Provision and service consumers use the same scope-value
policy once `decision:running-environments/resource-provisioning/handler-environment` is settled. Serves
`product:required-service-preconditions` and `product:ordered-preparation`.

## Destructive operation ownership (engineering:provision-effect-ownership)

Concurrent conflicting preparation/cleanup is serialized or refused, and interrupted operations preserve enough outcome
information to explain effects outside authoritative state. Structural teardown retains resource failures even when
continuing, and cannot infer provider cleanup success from a suppressed hook error. Serves
`product:declared-cleanup-scope`, `product:resource-teardown-integration`, and `product:preparation-partial-results`.

## Teardown options of `ws destroy` (engineering:destroy-teardown-options)

This epic registers `--no-provision-teardown` on `ws destroy` and adds `Options.provision_teardown` to
[destroy.md](../../parallel-workspaces/environment-lifecycle/spec/destroy.md), Domain, as its System table assigns; its
provision teardown phase also reads `Options.strict`, which `epic:agent-ready-operation/executable-extensions` adds with
`--strict` and on which this epic already depends. The same change sets `Forwarded.provision_teardown` of
[teardown.md](../../parallel-workspaces/nested-workspaces/spec/teardown.md), Forwarded, from its field, so each nested
child's `ws destroy` receives the option, which nested-workspaces leaves to the option's owner. Serves
`product:resource-teardown-integration` and `product:nested-teardown-preservation`.

Acceptance:

- A `ws destroy --force --no-provision-teardown` of an env holding a nested child starts the child's
  `winter ws destroy <env>` with `--force` and `--no-provision-teardown`, and one without the option starts it without.
- With `--strict`, a failed provision teardown ends the target with `StrictAbort` naming the provision phase.

## Ordered composition boundaries (engineering:bringup-phase-boundaries)

Composed bring-up carries phase outcomes across structural reconciliation, preparation, and workload launch, and admits
each phase only after its predecessor succeeds for that environment. Required infrastructure is distinguished from
workload launch; no readiness record or invalidation scheme is assumed until its owned decision is settled. Serves
`product:composed-environment-bringup`.
