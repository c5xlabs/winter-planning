---
name: local-page
depends-on:
  - apply-export
---

# The runner's Settings tab

Adds the Settings tab to the runner's local page ([runner-api.md](../spec/runner-api.md) §The local page). Delivers the
local-page half of `product:runner-settings-api` and `product:settings-in-force`.

## Change

- The Settings tab: host and runner settings with their lifetimes, the `in_force` line, an edit dialog per key for the
  superuser, workspaces and subscriptions, and history.

## Acceptance

- `web:unit-test`, `web:lint`, `web:typecheck`, `web:story-render`, and `web:client-drift` pass.
- `blizzard:ui` over editing a budget and reading `in_force` move after a step, and a guest seeing no edit control.
- `web:manual-panel`.
