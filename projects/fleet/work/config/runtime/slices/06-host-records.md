---
name: host-records
depends-on:
  - host-settings
---

# Workspaces and subscriptions as records

Makes the host's workspaces and subscriptions records with six verbs each, rebuilt by the host tick, and removes the
default workspace ([host-records.md](../spec/host-records.md) §Workspaces and subscriptions,
[runner-store.md](../spec/runner-store.md) §No default workspace). Delivers `product:host-records` and
`product:workspace-reshape`; shaped by `decision:config/runtime/no-default-workspace`.

## Change

- Their tables and seed; the six verbs on each.
- The `HostWorkspaces` holder every reader goes through, locks carried by id, and an empty registry.
- The host tick's rebuild of providers, samplers, and renewers; the host report built from the snapshot.
- The workspace prompt's routes write the record under the superuser check, and its table is deprecated
  (`engineering:deprecations-outlive-epic`).
- A runner naming a retired subscription runs as though it named none.
- No default workspace: the `workspace-local` fallback, `init`'s scaffolded workspace, and the `BZ_WORKSPACE_*`
  variables are gone, and the change note marks it breaking.
- Every test, harness profile, and script that leaned on the fallback or on `init`'s scaffolded workspace declares its
  workspace, or creates it through the API.
- Context: the domain's statement in `execution/hosts.md` §Bindings no longer halts a runner for a retired workspace or
  subscription; `bzh:configured-record` extends its scope to the host's records, whose change rows are
  `host_setting_changes` (`engineering:context-rules`).

## Acceptance

- `blizzard:unit-test` over each verb's refusals and the idempotent retire and enable.
- `blizzard:component-test` over:
  - a workspace repointed between two acquisitions with both taking the one lock;
  - a held environment released through the rebuilt provider;
  - every reader seeing a created workspace at the next tick;
  - a host with no workspace running and claiming nothing;
  - a retired workspace refusing its runner's next write until the binding is fixed, while its other bindings keep
    claiming;
  - a retired subscription leaving its samples while its runner works on unthrottled by it.
- `blizzard:service-test` against the mock hub seeing the host report change.
- `blizzard:harness-check`, `blizzard:store-contract`, `blizzard:cli-contract`, `web:client-drift`, and
  `blizzard:compat` pass.
