---
name: hub-legacy-retirement
depends-on:
  - hub-lifecycle-passes
code:
  - blizzard/src/blizzard/hub/composition.py::HubServices
  - blizzard/src/blizzard/hub/composition.py::TenantServices
  - blizzard/src/blizzard/hub/composition.py::TenantStores
---

# Close the hub's legacy registration

Closes the hub portion of spec [Mixed ownership](../spec/migration.md#flows), delivering `engineering:container-only`
and `engineering:rules-follow-wiring` under `decision:adopt-di/declared-cutover`. Available field/branch removal has
already happened with its last consumer.

## Change

- Remove empty hub legacy registration, aggregate types/assemblers and transition allowances after proving no consumers.
- Make the hub's area-root, no-bag, registered-construction and typed-accessor guards container-only.
- Bring hub harness rules/exemplars to their end state while explicitly retaining unmigrated runner/CLI guidance.

## Acceptance

- A source/inventory gate finds no hub legacy provider, aggregate construction or services-bag dependency; planted
  violations fail rather than being added to an allowlist.
- Hub roots contain area composition and lifecycle orchestration, not individual service registrations.
- Whole-graph and every-route tests, strict typing, relevant component/service tests and compatibility/config/CLI
  contracts pass unchanged.
- If an aggregate still has a live consumer, its owning migration change is incomplete and this closure does not pass;
  no builder is removed first while its bag remains needed.
