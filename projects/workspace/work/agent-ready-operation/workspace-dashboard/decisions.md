# Workspace dashboard decisions

## Contribution breadth (decision:agent-ready-operation/workspace-dashboard/contribution-scope)

**Decider:** product owner and engineering

**Open.** Choose which legacy command contributions and arbitrary custom screens become separately launched external
experiences, supported data-described views, or explicitly retired capabilities, beyond the badge/panel/action baseline;
this blocks finalizing `product:dashboard-contribution-outcomes` and requires an inventory of all five contribution
kinds rather than treating structured badges as complete replacement.

## Freshness boundary (decision:agent-ready-operation/workspace-dashboard/freshness-boundary)

**Decider:** product owner and engineering

**Open.** Choose coherent live reload across views and action targets or a snapshot boundary requiring reopen/restart
for keymaps and contributions, rather than legacy mixed-generation collaborators; this blocks finalizing
`product:dashboard-freshness` and action-target freshness in `product:dashboard-deliberate-actions`.

## Action concurrency and exit (decision:agent-ready-operation/workspace-dashboard/action-concurrency)

**Decider:** product owner and engineering

**Open.** Choose session-wide single-flight mutation or finer target/operation concurrency, and whether quit cancels
with diagnosed partial effects or hands supported work to a tracked continuation, preserving cross-process operation
safeguards either way; this blocks finalizing `product:dashboard-deliberate-actions`, not the prohibition on duplicate
conflicting mutations.

## Session error retention (decision:agent-ready-operation/workspace-dashboard/error-retention)

**Decider:** product owner and engineering

**Open.** Choose bounded session-only retention with visible eviction or explicitly requested export/persistence instead
of copying the unbounded in-memory log; this blocks finalizing retention in `product:dashboard-error-history` and does
not create a second global logging/tracing contract.

## Replace imported Python contributions (decision:agent-ready-operation/workspace-dashboard/contribution-replacement)

The dashboard replaces the in-process Python plugin seam with runtime-independent data contributions and external
actions under `decision:rust-replacement` and the project external-code boundary, rather than embedding Python,
importing plugins, or adding a Python compatibility host. This preserves supported user outcomes without committing to
Click, Rich, Textual, or their object contracts; the contribution-scope decision prevents silent capability loss.
