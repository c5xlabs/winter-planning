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
- Build graph, duplicate-owner, lifetime, alias-identity, role-key, default-factory, and import-boundary guard helpers.

## Acceptance

- `blizzard:unit-test` builds representative process/child/request and host/runner/operation graphs with strict
  validation; missing dependencies, duplicate owners, cycles, illegal lifetime dependencies, and automatic defaults
  fail.
- Thirty-two concurrent resolutions share one owned instance; two roles remain distinct; aliases share one finalizer.
- A resource failure unwinds partial acquisition; closing scopes preserves borrowed values and attempts all finalizers.
- `blizzard:typecheck` accepts valid adapter bindings and rejects fixture calls with a missing or incompatible binding;
  the translation waiver does not spread to root or area modules.
- Existing production roots and their behavior are unchanged.
