---
name: host-records
depends-on:
  - host-settings
code:
  - blizzard/src/blizzard/runner/composition.py::build_workspaces
  - blizzard/src/blizzard/runner/composition.py::host_report
  - blizzard/src/blizzard/runner/composition.py::credential_renewal_pass
---

# Host workspace and subscription records

Moves the host-records row in spec [Mixed ownership](../spec/migration.md#area-ownership), using
[Objects built from settings](../spec/scopes.md#objects-built-from-settings) and the host tick in
[Flows](../spec/scopes.md#flows). Delivers `engineering:area-wiring` and `engineering:ordered-teardown` under
`decision:adopt-di/after-runtime-config`.

## Change

- Register `HostWorkspaces` at host lifetime; it rebuilds its `WorkspaceRegistry` when a workspace record moves and
  keeps locks by `workspace_id`.
- Register `HostTick`'s `ExternalUsageSamplePass` and `CredentialRenewalPass` at host lifetime, present with zero
  subscription records in place of today's absent pass; each holds one sampler or renewer per record and rebuilds it
  when its record moves. The container holds none of the rebuilt objects.
- Register the workspace and subscription record services and the host report built per tick, and fill the `app.state`
  slots their routes read; the workspace prompt routes migrate in `runner-api-settings`, and the host workspace and
  subscription routes in `runner-api-host`.
- Remove the workspace and subscription construction from the host graph and its fields.

## Acceptance

- `blizzard:component-test`: a created workspace reaches every reader at the next host tick; a repointed workspace keeps
  its lock across the rebuild; a retired subscription stops being sampled while its samples stay; a host with no
  subscriptions runs both passes and samples nothing.
- `blizzard:service-test` against the mock hub sees the host report change after a record write.
- No provider holds a `WorkspaceRegistry`, sampler or renewer; `blizzard:typecheck` and `blizzard:unit-test` (the host
  graph/identity guards in `tests/guards`) pass.
