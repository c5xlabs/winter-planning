# Service orchestration decisions

## Provider activation (decision:running-environments/service-orchestration/provider-activation)

**Decider:** product owner and engineering

**Open.** Choose explicit-only versus implicit sole/all-provider activation, compatibility for undeclared provider
versions, and override precedence; this blocks finalizing `product:provider-binding`, not consistent validity checking
or explicit reporting of the selected set.

## Readiness enforcement (decision:running-environments/service-orchestration/readiness-enforcement)

**Decider:** product owner and engineering

**Open.** Choose launch-time preparation enforcement, whether evidence is persisted or checked live and how it is
invalidated, unprobed-service behavior, workspace coverage, and timeout composition with provider gates; this blocks
finalizing `product:truthful-service-readiness` and working-state acceptance for
`product:running-environments/resource-provisioning/composed-environment-bringup`, without claiming the Python baseline
has a persisted provision gate or accepting missing status as readiness success by accident.

## Log selection policy (decision:running-environments/service-orchestration/log-selection-policy)

**Decider:** product owner and engineering

**Open.** Choose aggregate backlog ordering, global versus per-service tail, follow backlog guarantees, and retained
behavior when providers lack timestamps or rotation history; this blocks finalizing `product:aggregate-service-logs`,
not fairness or cleanup of live streams.

## Built-in execution placement (decision:running-environments/service-orchestration/builtin-execution-placement)

**Decider:** engineering

**Open.** Choose execution placement for native built-ins while retaining the same Winter-owned semantics and
conformance as external providers; this blocks finalizing `engineering:service-provider-conformance`, not independent
delivery of the three provider epics or the rule that external extension code runs out of process.

## Declaration continuity (decision:running-environments/service-orchestration/declaration-continuity)

**Decider:** product owner and engineering

**Open.** Choose how startup-contributed services remain available for later ownership, observation, restart, logs, and
teardown, including changes or removal of their declaration; this blocks finalizing
`product:service-declaration-continuity` without prescribing a manifest schema or persistence mechanism.
