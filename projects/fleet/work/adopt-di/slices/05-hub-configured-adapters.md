---
name: hub-configured-adapters
external: github:paul-gross/blizzard#2355
depends-on:
  - hub-store-adapters
code:
  - blizzard/src/blizzard/hub/composition.py::build_live_config
  - blizzard/src/blizzard/hub/composition.py::build_config_authoring
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
  - blizzard/src/blizzard/hub/app.py::build_hosted_app
  - blizzard/src/blizzard/hub/api/chunk_events.py::ChunkChanged
  - blizzard/src/blizzard/hub/api/chunk_events.py::ChunkFrameState
  - blizzard/src/blizzard/hub/api/chunk_events.py::load_frame_states
  - blizzard/src/blizzard/hub/forge/readiness.py::CredentialReadiness
  - blizzard/src/blizzard/hub/forge/commit_links.py::LandedCommitLinks
  - blizzard/src/blizzard/hub/composition.py::HubCore
  - blizzard/src/blizzard/hub/domain/chunk/model.py::WorkRefLabel
  - blizzard/src/blizzard/hub/api/user_names.py::user_names
---

# Configured adapter wiring and consumers

Delivers the configured-adapter row of spec [Mixed ownership](../spec/migration.md#area-ownership) and its resource
ownership in spec [Scope ownership](../spec/scopes.md), serving `engineering:area-wiring`,
`engineering:ordered-teardown`, and `engineering:narrow-api-edge`; follows `decision:adopt-di/declared-cutover`.

## Change

- Move process settings/flags, live config, authoring/application, registry factories, forge budgets, commit resolution,
  and reference seams to their area's providers, using explicit factories for defaults. This slice owns the `HubConfig`
  source, `live_config` and the `pass_config` it supplies (`hub/app.py`), which `ActionPassDriver`, trace and egress
  config, the annotation floor, work-source close-writes, `TurnRunner` and `_bind_request_config` reuse.
- `ActionLoop`, `StepWake` and `ColorPick`, which `SeamCommitResolver` and `ConfigApplication` take, enter as legacy
  leaf providers; hub lifecycle adopts `ActionLoop` and `StepWake`, and projects adopts `ColorPick`.
- Narrow config, secret, forge, credential, repository and work-source handlers and choice/parameter helpers to their
  named services/read seams.
- Register `CredentialReadiness` through an explicit factory for its `budget` default, its clock as a registered
  dependency, and its event-log dependency as a leaf provider; register `credential_probes` with its override seam and
  the GitHub App installation tokens inside live config.
- Register `WorkRefLabel` (today `work_sources.label`, `TenantServices.work_ref_label`) as the work-source area's single
  owner; it replaces the legacy leaf provider that chunk execution's claim and apply, the fleet API, and trace export's
  `StepHydration` would otherwise take.
- Narrow the `user_names` helper to the user read seam it uses, in place of `TenantServices`.
- Move the work-source and forge registry factories out of `build_hosted_app` into their areas. The work-source
  registry's inputs not yet adopted, `TenantStores.work_item_edits`, `TenantStores.garden_proposal_resolution` and
  `HubCore.users`, enter as legacy leaf providers that chunk operations, garden authoring and hub identity adopt.
- Register `hub_key_provider`'s key provider and `secret_cipher` at process lifetime, shared by config authoring, live
  config and secret rotation.
- Register `LandedCommitLinks` over the registered project-link, repository and forge record stores; the chunk views and
  chunks API and hub lifecycle's `CloseIntentDrainer` reuse it.
- Own the narrowing of the chunk-event helpers, since the work-source API is their first migrated caller. `ChunkChanged`
  (a value built per write around a chunk id) and `ChunkFrameState` (a plain record) stay ordinary values, and are not
  registered; `ChunkChanged` and `load_frame_states` take typed arguments, the chunk-facts, chunk-record, route, graph
  and runner-registry-name reads they load and the tenant's broker publisher, in place of `TenantServices`. At most one
  typed loader over those reads is registered. The chunk ports enter as legacy leaf providers that chunk operations
  adopts; urgency and later callers reuse these helpers, and unmigrated callers pass the values from the bag.
- The admin-settings routes take the named services of the borrowed `HubSettingsGraph`, whose aliases the store-adapter
  change registers.
- Retire migrated fields/branches when unused; preserve configured-client refresh behavior and owner-loop closure.

## Acceptance

- Component tests cover config edit/refresh and adapter reuse; one adapter's aliases share identity and lifetime.
- Eviction/shutdown closes owned configured clients on the existing owner while borrowed clients remain open.
- No affected handler/helper takes a services bag; remaining bag fields retain canonical identity.
- Chunk-frame publication from the work-source API and the urgencies API carries the same frames as today
  (`blizzard:sse-contract`).
- `blizzard:config-contract`, `blizzard:component-test`, `blizzard:service-test`, `blizzard:typecheck`, and the
  graph/route/ownership gates pass; no config or wire shape changes.
