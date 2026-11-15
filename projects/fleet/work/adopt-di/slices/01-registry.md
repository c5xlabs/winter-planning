---
name: registry
---

# Typed registry and translation

Builds the registration interface and synchronous translation in spec [Area registration](../spec/registry.md),
delivering `engineering:pin-dishka-one`, `engineering:strict-waiver-at-translation`, `engineering:role-is-a-type`, and
`engineering:dishka-at-edge`; follows `decision:adopt-di/synchronous-resolution`. No production graph is activated here.

## Change

- Provide typed registration handles, port aliases, factories, resources, context values, and process-specific
  lifetimes.
- Pin non-deprecated Dishka 1.x APIs; keep overrides and the strict waiver outside area wiring.
- Provide the test-only override builder from spec [Area registration](../spec/registry.md#interfaces), separate from
  the area's registry interface.
- Build graph, duplicate-owner, lifetime, alias-identity, role-key, default-factory, and import-boundary guard helpers.

## Acceptance

- `blizzard:unit-test` builds representative process/child/request and host/runner/operation graphs with strict
  validation; missing dependencies, duplicate owners, cycles, illegal lifetime dependencies, and automatic defaults
  fail.
- Thirty-two concurrent resolutions share one owned instance; two roles remain distinct; aliases share one finalizer.
- A resource failure unwinds partial acquisition; closing scopes preserves borrowed values and attempts all finalizers.
- Override tests replace an existing factory or value before validation, retain the target's key/lifetime and alias
  identity, and leave supplied replacement values open at scope closure.
- Unknown override targets and replacement after container startup are refused; the area registry exposes no replacement
  operation, and an unreplaced duplicate production owner still fails validation.
- `blizzard:typecheck` accepts valid adapter bindings and rejects fixture calls with a missing or incompatible binding;
  the translation waiver does not spread to root or area modules.
- Existing production roots and their behavior are unchanged.
