---
name: registry
external: github:paul-gross/blizzard#2353
---

# Typed registry and translation

Builds the registration interface and synchronous translation in spec [Area registration](../spec/registry.md),
delivering `engineering:pin-dishka-one`, `engineering:strict-waiver-at-translation`, `engineering:role-is-a-type`,
`engineering:dishka-at-edge`, `engineering:wiring-modules-apart`, and `engineering:explicit-factory-for-defaults`;
follows `decision:adopt-di/synchronous-resolution`. No production graph is activated here.

## Change

- Provide typed registration handles, `port(...)` as the only `Port` constructor with `Port` declared invariant in its
  type parameter, port aliases, factories, resources, context values, `Registry.absent` registrations carrying their
  refusal label (resolving one raises `ServiceAbsent(label)`), `Registry.optional` registrations of the key `T | None`,
  and process-specific lifetimes.
- Add `dishka>=1.10,<2` to `pyproject.toml` and use only its non-deprecated API; keep overrides and the strict waiver
  outside area wiring.
- Provide the guarded synchronous resolver: owner state guards are shared for resolution and exclusive for closing, and
  a resolution through a closing or closed owner raises `ScopeClosed(lifetime, owner)`, carrying the closed owner's
  lifetime and id. Its `get(dependency_type, component=None)` takes `component` positional-or-keyword, as dishka's
  `Container.get` does, and forwards it.
- Provide the test-only override builder from spec [Area registration](../spec/registry.md#interfaces), separate from
  the area's registry interface.
- Build graph, duplicate-owner, lifetime, alias-identity, role-key, default-factory, and import-boundary guard helpers.

## Acceptance

- `blizzard:unit-test` builds representative process/child/request and host/runner/operation graphs with strict
  validation; missing dependencies, duplicate owners, cycles, illegal lifetime dependencies, and automatic defaults
  fail.
- Thirty-two concurrent resolutions share one owned instance and never wait on a parent's guard; two roles remain
  distinct; aliases share one finalizer; a resolution racing a close raises `ScopeClosed`.
- Each process's translation rejects a lifetime kind it does not use (the hub rejects `RUNNER`, the host rejects
  `TENANT`); a client-only `COMMAND`-lifetime graph builds, resolves and closes its command-owned resources on exit and
  on an exception.
- A `Port` whose annotation differs from the key it is registered under is rejected at translation.
- The suite's `DeprecationWarning`-as-error filter (`pyproject.toml`) runs over every dishka call the translation makes,
  so a deprecated form fails `blizzard:unit-test`.
- A resource failure unwinds partial acquisition; closing scopes preserves borrowed values and attempts all finalizers.
- Override tests replace an existing factory or value before validation, retain the target's key/lifetime and alias
  identity, and leave supplied replacement values open at scope closure.
- Unknown override targets and replacement after container startup are refused; the area registry exposes no replacement
  operation, and an unreplaced duplicate production owner still fails validation.
- A `blizzard:unit-test` runs pyright over planted fixtures, as `tests/guards/test_strict_types_scan.py` does: it
  accepts valid adapter bindings and rejects a missing or incompatible binding and an `alias` whose port and target
  roles differ (`Port[IReadFindingRepository]` against a `Ref[IWriteFindingRepository]`); `blizzard:typecheck` passes
  over the production modules, and the translation waiver does not spread to root or area modules.
- Resolving an `absent` registration raises `ServiceAbsent` carrying its label; whole-graph validation passes with it
  present.
- An `optional` registration resolves `T | None` to the target's instance when the root has it, the same instance `T`
  resolves to, and to `None` when the root registers `T` absent.
- A `ScopeClosed` names the closed owner's lifetime and id.
- An `inject_sync`-wrapped handler resolves a component-keyed dependency through the guarded resolver, which dishka's
  injector calls with `component` positionally; a keyword call resolves the same key.
- Existing production roots and their behavior are unchanged.
