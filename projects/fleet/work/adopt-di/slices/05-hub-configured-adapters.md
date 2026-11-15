---
name: hub-configured-adapters
depends-on:
  - hub-store-adapters
code:
  - blizzard/src/blizzard/hub/composition.py::build_live_config
  - blizzard/src/blizzard/hub/composition.py::build_config_authoring
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
---

# Configured adapter wiring and consumers

Delivers the configured-adapter row of spec [Mixed ownership](../spec/migration.md#area-ownership) and its resource
ownership in spec [Scope ownership](../spec/scopes.md), serving `engineering:area-wiring`,
`engineering:ordered-teardown`, and `engineering:narrow-api-edge`; follows `decision:adopt-di/declared-cutover`.

## Change

- Move process settings/flags, live config, authoring/application, registry factories, forge budgets, commit resolution,
  and reference seams to their area's providers, using explicit factories for defaults.
- Narrow config, secret, forge, credential, repository and work-source handlers and choice/parameter helpers to their
  named services/read seams.
- Retire migrated fields/branches when unused; preserve configured-client refresh behavior and owner-loop closure.

## Acceptance

- Component tests cover config edit/refresh and adapter reuse; one adapter's aliases share identity and lifetime.
- Eviction/shutdown closes owned configured clients on the existing owner while borrowed clients remain open.
- No affected handler/helper takes a services bag; remaining bag fields retain canonical identity.
- Relevant config/API tests, `blizzard:typecheck`, and graph/route/ownership gates pass; no config or wire shape
  changes.
