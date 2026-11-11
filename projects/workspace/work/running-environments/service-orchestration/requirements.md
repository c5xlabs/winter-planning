# Service orchestration requirements

For `epic:running-environments/service-orchestration`.

## Select capable providers (product:provider-binding)

A person can see which providers own service operations and choose compatible implementations without silently enabling
unsupported work. Binding policy is settled by
`decision:running-environments/service-orchestration/provider-activation`.

Acceptance:

- Explicit single/multiple bindings, available implicit candidates, local development overrides, and invalid or unbound
  states are inspectable without starting services.
- Missing entrypoints, incompatible declared versions, and malformed provider declarations are identified consistently
  for explicit and implicit resolution; skipped candidates do not disappear from a complete result.
- An override has visible precedence, applies only to that invocation, and does not rewrite workspace bindings.
- Built-in and external implementations honor the same Winter-owned operations and conformance expectations; unsupported
  selections or features produce an explicit refusal rather than backend flag passthrough.

## Target services precisely (product:precise-service-selection)

An agent can target one service, a scope, or matching services across environments without stopping unrelated services.

Acceptance:

- Segment-aware scope/service globs, multiple operands, and bare-scope selection work consistently for control, status,
  restart, and logs; wildcard feature selection excludes workspace scope unless explicitly selected.
- Mutations require an explicit target; read-only status can inspect all configured scopes, including stopped ones.
- Multi-provider ownership is unambiguous; conflicting claims identify both providers before affected mutation.
- Several filters for one scope remain their union, never a whole-scope operation caused by dispatch simplification.
- Invalid restart operands are reported before any restart; unresolvable selections never imply broader work.
- A provider unable to enumerate cross-environment selection is refused clearly or receives correctly resolved targets,
  rather than silently dropping wildcard scopes.

## Explain service lifecycle effects (product:service-control-outcomes)

The engineer can start, stop, and restart selected services with clear shared infrastructure and partial effects.

Acceptance:

- Feature startup ensures workspace infrastructure first; failure there remains unsuccessful even when requested feature
  starts are still attempted. Explicit workspace-only operations do not recurse into feature scopes.
- Feature shutdown leaves shared services running; shared shutdown requires explicit workspace selection.
- Requested startup fan-out stops on its first failed target, while shutdown attempts remaining independent targets;
  outcomes identify completed, failed, skipped, and unattempted work without pretending to roll back earlier starts.
- Restart leaves unrelated services running, states whether it reloads runtime values, and does not imply readiness
  verification that has not occurred.
- Provider shutdown errors, missing prerequisites, and cancellation cannot be presented as verified cleanup success.

## Observe state and readiness honestly (product:truthful-service-readiness)

A person can distinguish running, stopped, unhealthy, unprobed, and unobservable services before trusting a scope.
Readiness and preparation enforcement are settled by
`decision:running-environments/service-orchestration/readiness-enforcement`.

Acceptance:

- Status identifies scope and service with state, health, known ports, runtime handle, log location, and start time when
  available; unavailable fields and unsupported observations are explicit, not fabricated.
- Partial provider failure preserves valid observations but marks their incompleteness and returns an unsuccessful
  aggregate observation when promised coverage was not obtained.
- A wait names its selected targets, finite timeout, and unresolved services; missing, failed, or malformed observation
  is not counted as readiness success unless a deliberate decision defines and visibly labels a weaker outcome.
- Unprobed services follow an explicit rule; provider-native startup gates and Winter's requested wait disclose their
  different scope and guarantees rather than silently granting equivalent readiness.
- A workload cannot be reported safely prepared merely because no persisted provision record exists or no unhealthy
  service was observed. The chosen enforcement policy exposes unmet preparation/readiness before claiming usable state.

## Read attributable logs fairly (product:aggregate-service-logs)

The engineer can inspect or follow several services without one quiet or long-running stream hiding the others.

Acceptance:

- Single-service output can be read without redundant prefixes; aggregate output identifies scope/service and optionally
  timestamps, with diagnostics kept off the result stream.
- Tail counts, all-backlog selection, relative/absolute time bounds, and inclusive boundaries have documented meanings;
  missing timestamps expose partial filtering instead of inventing times.
- Follow serves every selected live stream fairly within and across providers; one provider's exit does not terminate
  its peers, and nonzero exits remain visible in the final result.
- Backlog ordering, global versus per-service tail behavior, and follow backlog limitations are explicit under
  `decision:running-environments/service-orchestration/log-selection-policy`.
- Cancellation and broken output pipes clean up invocation-owned followers and descendants without stopping the observed
  services; interrupted follow is distinguishable from ordinary completion.

## Discover declared services reliably (product:service-declaration-continuity)

A person can declare services in the workspace or extensions and still find them after startup or a later invocation.

Acceptance:

- Service catalog membership is separate from runtime observation and ownership; lookup failure is not an empty trusted
  catalog or a known-stopped service without a diagnostic.
- Workspace and extension contributions retain scope/source attribution, and collisions or incompatible provider needs
  are reported before affected launches.
- Services introduced at startup remain discoverable for later status, restart, logs, and teardown, or the unsupported
  continuity is explicitly refused before accepting the declaration.
- Provider-specific capabilities and prerequisites are discoverable; choosing a runtime cannot silently discard health,
  logging, dependency, interaction, or environment features it does not support.
