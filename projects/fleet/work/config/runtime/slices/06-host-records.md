---
name: host-records
depends-on:
  - host-settings
code:
  - blizzard/src/blizzard/runner/composition.py::build_workspaces
  - blizzard/src/blizzard/runner/composition.py::host_report
  - blizzard/src/blizzard/runner/composition.py::credential_renewal_pass
  - blizzard/src/blizzard/runner/environments/workspaces.py::WorkspaceRegistry
  - blizzard/src/blizzard/runner/config.py::DEFAULT_WORKSPACE_ID
  - blizzard/src/blizzard/runner/host_config.py::HostConfig.scaffold
  - blizzard/src/blizzard/runner/api/workspace_prompt.py::replace_workspace_prompt
  - blizzard/src/blizzard/runner/api/workspace_prompt.py::clear_workspace_prompt
  - blizzard/src/blizzard/runner/api/host.py::list_workspaces
  - blizzard/src/blizzard/runner/api/host.py::list_subscriptions
---

# Workspaces and subscriptions as records

Makes the host's workspaces and subscriptions records with six verbs each, rebuilt by the host tick, and removes the
default workspace ([host-records.md](../spec/host-records.md) §Domain, §Workspaces and subscriptions, §The workspace
prompt; [runner-store.md](../spec/runner-store.md) §Seeding steps 2–3, §No default workspace;
[passes.md](../spec/passes.md) §The host tick). Delivers `product:host-records` and `product:workspace-reshape`; shaped
by `decision:config/runtime/no-default-workspace`, the change note marking it breaking.

## Change

- `host_workspaces`, `host_subscriptions`, and their facts; their seed; the six verbs on each, routes and CLI.
- `HostWorkspaces`, the holder every reader goes through; locks carried by `workspace_id`; an empty registry served.
- The host tick's rebuild of providers, samplers, and renewers; the host report built per tick.
- The workspace prompt's routes write the record under `require_superuser`; the `workspace_prompt` table deprecated
  (`engineering:deprecations-outlive-epic`).
- A binding to a retired workspace no longer halts its runner; a runner naming a retired subscription runs as though it
  named none.
- The `workspace-local` fallback, `init`'s scaffolded workspace, and the `BZ_WORKSPACE_*` variables removed.
- Every test, harness profile, and script leaning on the fallback or on `init`'s workspace declares its workspace or
  creates it through the API.
- Context: `execution/hosts.md` §Bindings no longer halts a runner for a retired workspace or subscription;
  `bzh:configured-record` extends to the host's records, whose change rows are `host_setting_changes`
  (`engineering:context-rules`).

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
