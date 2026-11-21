---
name: hub-store-adapters
depends-on:
  - hub-request-pilot
code:
  - blizzard/src/blizzard/hub/composition.py::build_hub_core
  - blizzard/src/blizzard/hub/composition.py::build_tenant_stores
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
  - blizzard/src/blizzard/hub/composition.py::build_services
  - blizzard/src/blizzard/hub/composition.py::build_live_config
  - blizzard/src/blizzard/hub/composition.py::open_hub_store
  - blizzard/src/blizzard/hub/cli/runtime.py::host
  - blizzard/src/blizzard/hub/store/internal/chunk_store_factory.py::build_chunk_stores
---

# Hub store registrations

Moves the scoped-store ownership row of spec [Mixed ownership](../spec/migration.md#area-ownership) into adapter wiring,
delivering `engineering:area-wiring`, `engineering:one-instance-per-port`, and
`engineering:behavior-preserving-landings` under `decision:adopt-di/adoption-batch-size`.

## Change

- Register every remaining hub/store adapter and transaction primitive: those the three listed roots construct, and
  `EgressStore`, `TenantAdministrationStore` and `TenantTeardownStore` (`build_services`), `ConfigRevisionsStore`
  (`build_live_config`) and `build_chunk_stores`, among them `InvitationStore` and `HubScopedReads.invitations`; retain
  the finding registrations already adopted. `TraceStore` moves with trace export, since its `StepHydration` reads the
  work-source registry. The `hub/auth/internal` repositories and `UserRepository` move with hub identity;
  `TenantAdministrationStore` takes `IdentityTransactions` as a legacy leaf provider here, which hub identity adopts.
- The hub settings store is not registered here: `hub host` builds `HubSettingsGraph` before any container exists
  (`build_hub_settings`), and its settings ports alias the members of that borrowed graph. A hub app built without
  settings (`create_app(settings=None)`, `create_app_for_export`) registers those settings ports with `Registry.absent`
  and the `hub_store_unavailable` label, so the admin-settings routes keep answering `hub_store_unavailable_problem`'s
  `503` through the hub's `ServiceAbsent` handler instead of `admin_settings.graph`'s own check.
- Take the engine and `HubStoreConnections` as borrowed context from `hub host` ([Scope ownership](../spec/scopes.md));
  no provider opens a hub store. `test_hub_store_wiring_goes_through_composition`, which the request pilot retargets to
  the container, keeps passing unchanged.
- Replace their old constructor branches with canonical resolutions; services embedded in `TenantStores` remain with
  their concept owners, not the store package.
- Remove proved-dead fields only after checking API, helpers, and background consumers
  (`engineering:dead-members-deleted`).

## Acceptance

- Each scoped-store constructor has one provider owner; alias and legacy-store field identity checks pass.
- Unit/component tests preserve tenant filtering, transaction boundaries, one borrowed engine and connection set across
  the hosted composition that stays open after the container closes, and the bridged tenant cache's lifetime.
- The graph and current routes resolve; no new concrete store construction remains in the listed roots.
- No second hub settings store is constructed: the settings ports and `HubSettingsGraph`'s store are one instance.
- On an app built without settings, the admin-settings routes answer `hub_store_unavailable_problem`'s `503` with the
  payload the refusal inventory guards pin (`blizzard:component-test`).
- `blizzard:store-contract`, `blizzard:component-test`, `blizzard:service-test` and `blizzard:typecheck` pass with
  unchanged API behavior.
