# Resource provisioning decisions

## Shared cleanup policy (decision:running-environments/resource-provisioning/shared-cleanup-policy)

**Decider:** product owner and engineering

**Open.** Choose whether shared handlers require separate selection/approval, how multi-environment runs deduplicate or
repeat them, and which destructive actions require confirmation versus explicit unattended authorization; this blocks
finalizing `product:declared-cleanup-scope` and `product:resource-teardown-integration` without permitting hidden shared
effects.

## Service precondition policy (decision:running-environments/resource-provisioning/service-precondition-policy)

**Decider:** product owner and engineering

**Open.** Choose running-state versus health requirements, targeted-service versus whole-scope startup, post-start
verification, and ownership of services left running after a check; this blocks finalizing
`product:required-service-preconditions` but not independent stage execution.

## Handler environment (decision:running-environments/resource-provisioning/handler-environment)

**Decider:** product owner and engineering

**Open.** Choose whether provisioning retains legacy identity-only injection or receives resolved scope bands, including
lookup frequency and explicit inherited-environment boundaries; this blocks finalizing environment behavior under
`product:ordered-preparation`, not its stage ordering or action vocabulary.
