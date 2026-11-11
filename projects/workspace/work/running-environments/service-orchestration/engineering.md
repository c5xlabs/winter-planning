# Service orchestration engineering requirements

For `epic:running-environments/service-orchestration`.

## What already holds

`engineering:provider-contract` and `engineering:external-code-boundary` establish Winter-owned semantics and external
process isolation without deciding execution placement for native built-ins. `engineering:concurrent-state-safety` and
`engineering:observability-isolation` govern authoritative updates and trace-free managed launches.

## Uniform conformance (engineering:service-provider-conformance)

One capability definition governs built-in and external provider selection, refusal, state, health, catalog, and logs.
Conformance checks establish observable behavior beyond action recognition or argument echo; process-per-built-in is not
a requirement. Serves `product:provider-binding` and `product:service-declaration-continuity`.

## Non-widening dispatch (engineering:service-selection-authority)

Dispatch preserves scope/service selection and validates ownership before destructive effects; enumeration and
post-observation filters cannot replace pre-mutation target precision. Deterministic aggregation retains incomplete
provider coverage and target outcomes, rather than deriving success solely from valid output fragments. Serves
`product:precise-service-selection`, `product:service-control-outcomes`, and `product:truthful-service-readiness`.

## Independent observation and launch (engineering:service-observation-boundary)

Observation does not run configured value lookups, install dependencies, repair registry identity, or start services.
Launch consumes complete scope computation; readiness retains observation failures and enforces a finite agreed budget
without converting an absent document into an empty healthy set. Serves `product:truthful-service-readiness` and
`product:service-control-outcomes`.

## Bounded streaming and cleanup (engineering:service-stream-lifecycle)

Log aggregation bounds buffering and applies cancellation/backpressure to every acquired follower; cleanup preserves the
primary failure while terminating and reaping invocation-owned processes, not managed workloads. Fairness covers streams
within a provider as well as providers; timestamp absence remains data, not a parsing success fiction. Serves
`product:aggregate-service-logs`.
