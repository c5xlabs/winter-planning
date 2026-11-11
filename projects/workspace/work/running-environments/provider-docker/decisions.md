# Docker provider decisions

## Port assignment (decision:running-environments/provider-docker/port-assignment)

**Decider:** product owner and engineering

**Open.** Choose compatibility for declaration-order offsets, service-name normalization, port-band exhaustion, and
assignment changes when services are reordered; this blocks finalizing `product:docker-port-values`, without promising
that every arbitrary compose port is automatically isolated.

## Readiness coverage (decision:running-environments/provider-docker/readiness-coverage)

**Decider:** product owner and engineering

**Open.** Choose selected-service, dependency, and replica coverage plus composition of native startup waiting with the
caller timeout and Winter readiness policy; this blocks finalizing `product:docker-health-and-logs`, not treating failed
observation and starting containers honestly.

## Service declaration support (decision:running-environments/provider-docker/service-declaration-support)

**Decider:** product owner and engineering

**Open.** Choose the supported relationship between Winter-declared services, compose-only services, replicas, and
startup-contributed declarations, with explicit refusal of unsupported shapes; this blocks finalizing
`product:docker-capability-limits` without inventing a compose-generation schema.
