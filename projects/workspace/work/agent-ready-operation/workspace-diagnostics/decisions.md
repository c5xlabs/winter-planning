# Workspace diagnostics decisions

## Degraded diagnostic startup (decision:agent-ready-operation/workspace-diagnostics/degraded-startup)

**Decider:** product owner and engineering

**Open.** Choose a clean unsuccessful startup report or a degraded diagnostic run that can inspect native/configuration
facts before full configuration loads; this blocks finalizing `product:diagnostic-startup-failure` and avoids promising
the legacy parse probe can run when constructing it already requires valid configuration.

## Contribution limits (decision:agent-ready-operation/workspace-diagnostics/contribution-limits)

**Decider:** product owner and engineering

**Open.** Choose shared finite defaults or owner-configurable finite execution/output bounds for health programs,
including how cancellation is reported, rather than preserving unbounded contribution calls; this blocks finalizing the
failure behavior of `product:contributed-health-observations`.
