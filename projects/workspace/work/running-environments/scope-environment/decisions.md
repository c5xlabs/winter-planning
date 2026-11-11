# Scope environment decisions

## Resolution compatibility (decision:running-environments/scope-environment/resolution-compatibility)

**Decider:** product owner and engineering

**Open.** Choose compatibility for managed-variable redeclarations, restricted workspace visibility, imported-key
precedence, intermediate reads before later imports, scalar conversion, and masked multi-value references; this blocks
finalizing `product:layered-runtime-values` and `product:offline-env-inspection`, not preserving explicit failure.

## Lookup execution policy (decision:running-environments/scope-environment/lookup-execution-policy)

**Decider:** product owner and engineering

**Open.** Choose lookup frequency and snapshot/refresh boundaries across scope dispatch, preflight, actual process
launch, and restart, and whether any explicit alternative to the 30-second non-interactive baseline is supported; this
blocks finalizing `product:external-value-resolution` and its provider integrations without allowing lookup execution on
routine status polls.

## Persistent value exposure (decision:running-environments/scope-environment/persistent-value-exposure)

**Decider:** product owner and engineering

**Open.** Choose which provider-generated artifacts may contain resolved values, with visibility, lifetime, and access
rules for required probe or runtime material; this blocks finalizing `product:private-runtime-values` and the affected
provider constraints rather than accepting the legacy claim that no generated configuration contains resolved values.
