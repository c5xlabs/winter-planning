---
name: host-records
external: github:paul-gross/blizzard#2481
depends-on:
  - host-settings
code:
  - blizzard/src/blizzard/runner/composition.py::HostRecordsReconcile
  - blizzard/src/blizzard/runner/environments/workspaces.py::HostWorkspaces
  - blizzard/src/blizzard/runner/composition.py::build_host_record_services
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

- Register `HostRecordsReconcile` at host lifetime as the stable owner of the record rebuilds, `optional` (`None` for a
  host keeping no records, as today), over the registered `HostSettingsReader`, `HostWorkspaces` and `HostTickInForce`
  in place of the whole `HostProcess`. It sets the rebuilt `WorkspaceRegistry` in `HostWorkspaces`, which keeps locks by
  `workspace_id`, and the rebuilt `HostTick` and served subscriptions in `HostTickInForce`. Register `HostWorkspaces` at
  host lifetime. The container holds none of the rebuilt objects.
- Keep the delivered order ([Flows](../spec/scopes.md#flows)): the records reconcile runs before the runner reconcile
  and the tick in every host pass, and before any runner child opens at boot.
- Register the workspace and subscription record services (`build_host_record_services`) and the host report built per
  tick, and fill the `app.state` slots their routes read; the workspace prompt routes migrate in `runner-api-settings`,
  and the host workspace and subscription routes (`runner/api/host_records.py`) in `runner-api-host`.
- Remove the workspace, subscription and reconcile construction from `build_host_process` and the `HostProcess`
  `workspaces` and `records_reconcile` fields. The stopped-host record paths (`seed_host_records`, `read_recorded_host`)
  move with their commands in `cli-runner-dual-mode`.

## Acceptance

- `blizzard:component-test`: a created workspace reaches every reader at the next host tick; a repointed workspace keeps
  its lock across the rebuild; a retired subscription stops being sampled while its samples stay; a host with no
  subscriptions samples nothing; a failed rebuild keeps the workspaces and subscriptions in force.
- `blizzard:component-test`: a workspace retired while its runner resumes releases its provider before that runner's
  first pass, at boot and on a running host.
- `blizzard:service-test` against the mock hub sees the host report change after a record write.
- No provider holds a `WorkspaceRegistry`, `HostTick`, sampler or renewer; `blizzard:typecheck` and `blizzard:unit-test`
  (the host graph/identity guards in `tests/guards`) pass.
