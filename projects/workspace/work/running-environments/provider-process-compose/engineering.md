# Process-compose provider engineering requirements

For `epic:running-environments/provider-process-compose`.

## What already holds

`engineering:provider-contract`, `engineering:external-code-boundary`, and `decision:rust-replacement` bind the complete
first-party workflow without selecting built-in execution placement. `engineering:cli-runtime-baseline` supplies common
CLI targets; `decision:supported-systems` owns supervisor compatibility and prerequisites on those targets.
Process-compose remains the external supervisor rather than a Winter daemon requirement.

## Native supervisor integration (engineering:process-compose-native-control)

Native control and maintained helper behavior meet the common provider conformance without Python delegation; supervisor
version differences cannot silently change startup, update, restart, or readiness guarantees. Serves
`product:process-compose-instances`, `product:process-compose-launch`, and `product:process-compose-feature-limits`.

## Recoverable instance state (engineering:process-compose-instance-authority)

Instance endpoint selection and authoritative updates preserve scope isolation under concurrency, interruption, and long
paths; a missing root is not recreated by inspection or recovery. Signaling authority survives daemon death only where
workspace/scope ownership and process identity can be validated against reuse; missing evidence yields refusal, not a
broader session sweep. Serves `product:process-compose-instances` and `product:process-compose-recovery`.

## Runtime value exposure (engineering:process-compose-rendered-values)

Generated launch definitions separate definition identity from resolved runtime values so unchanged services do not
restart merely because a lookup refreshed; resolved probe material follows the explicitly chosen persistence policy.
Dependency readiness and probe execution consume declared effective environment semantics rather than assuming shell
mapping and probe interpolation are equivalent. Serves `product:process-compose-launch`.

## Observation and follow boundaries (engineering:process-compose-observation-fidelity)

Control endpoint failure, malformed supervisor output, terminal state, and known-stopped state remain distinct. Log
following handles supported file lifecycle changes and incomplete lines without blocking another selected scope, and
bounded follower cleanup never signals supervised workloads. Serves `product:process-compose-observation`.
