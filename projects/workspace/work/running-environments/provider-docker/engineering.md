# Docker provider engineering requirements

For `epic:running-environments/provider-docker`.

## What already holds

`engineering:provider-contract`, `engineering:external-code-boundary`, and `decision:rust-replacement` apply to this
independently deliverable provider. `engineering:cli-runtime-baseline` supplies common CLI targets;
`decision:supported-systems` governs provider compatibility and Docker daemon/Compose prerequisites rather than a
separate CLI platform baseline.

## Native lifecycle coherence (engineering:docker-native-control)

Native first-party paths use one effective compose-scope and ownership interpretation across ordinary actions and
environment destroy; no maintained Python helper or obsolete shell-field parser remains required. Backend command
failure and parse failure remain distinguishable from known-stopped state. Serves `product:docker-compose-scopes` and
`product:docker-targeted-lifecycle`.

## Container resource authority (engineering:docker-resource-ownership)

Project, scope, and selected-service identity bound every effect; partial shutdown cannot invoke whole-project removal.
Port substitutions are validated against agreed capacity/name rules and reported separately from observed published
ports. Volume deletion requires the declared destructive authority rather than following container shutdown implicitly.
Serves `product:docker-port-values` and `product:docker-targeted-lifecycle`.

## Complete backend observation (engineering:docker-observation-coverage)

Native readiness and log aggregation conform to Winter's coverage, timeout, fairness, and interruption semantics across
selected services and agreed replica units. Backend output variations do not turn an observation error into an empty
successful observation, and follower cleanup does not signal containers. Serves `product:docker-health-and-logs`.

## Explicit declaration support (engineering:docker-declaration-conformance)

Capability validation identifies unsupported shared-manifest fields and continuity before launch; acceptance of an
action word is insufficient conformance evidence. Platform-dependent daemon access and port behavior remain explicit
verification dimensions. Serves `product:docker-capability-limits`.
