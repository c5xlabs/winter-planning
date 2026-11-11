# Workspace dashboard engineering requirements

For `epic:agent-ready-operation/workspace-dashboard`.

## Coherent observation generations (engineering:dashboard-observation-consistency)

Accepted dashboard updates bind one effective configuration generation to its target identities and freshness metadata;
older completions cannot overwrite newer accepted state or resolve an action against a different generation unnoticed.
Read-only CLI and dashboard observations share the owning capabilities' meaning, not independently inferred status.
Serves `product:dashboard-workspace-overview`, `product:dashboard-freshness`, and `product:dashboard-agent-inspection`.

## Bounded work and teardown (engineering:dashboard-work-lifecycle)

Refresh and detail work have bounded concurrency and cancellation-aware completion; detached views and a closed terminal
receive no late UI updates. Contributor failure is isolated at its owner boundary, and rendered repository content is
data rather than executable or implicitly interpreted control content. Serves `product:dashboard-repository-detail`,
`product:dashboard-freshness`, and `product:dashboard-keyboard-control`.

## Local error evidence (engineering:dashboard-error-accounting)

Captured errors preserve structured causes and target attribution independently of notification deduplication;
deduplication cannot suppress the retained occurrence. Retention and truncation follow
`decision:agent-ready-operation/workspace-dashboard/error-retention`, and clear removes both visible history and its
notification-suppression state without affecting operation state. Serves `product:dashboard-error-history`.

## Runtime-independent contributions (engineering:dashboard-data-authority)

Contribution observations and view descriptions cross the external-code boundary as data with source attribution, not
live runtime objects, callable handlers, imported command objects, or UI classes. Applying malformed returned data
cannot alter native state or claim another contributor's identity; this does not sandbox the program's own effects. The
replacement accounts separately for legacy commands, screens, panels, decorators, and actions. Serves
`product:dashboard-contribution-outcomes`.

## Mutations retain operation authority (engineering:dashboard-action-authority)

Dashboard actions use the same target validation and conflicting-update safeguards as the corresponding non-interactive
operation, including conflicts with another process; a UI-local lock alone does not establish authoritative safety.
Terminal handoff, non-interactive input, cancellation, and partial completion preserve the selected operation's actual
state rather than inventing rollback or completion when the view closes. Serves `product:dashboard-deliberate-actions`.
