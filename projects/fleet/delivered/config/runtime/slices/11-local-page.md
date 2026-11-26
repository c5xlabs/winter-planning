---
name: local-page
external: github:paul-gross/blizzard#2135
depends-on:
  - host-settings
  - host-records
code:
  - blizzard/web/projects/runner/src/app/shell/app.routes.ts::routes
---

# The runner's Settings tab

Adds the Settings tab to the runner's local page ([runner-api.md](../spec/runner-api.md) §The local page). Delivers the
local-page half of `product:runner-settings-api` and `product:settings-in-force`.

## Change

- The Settings tab beside Board, Events, and Host: host and runner settings with their lifetimes and the `in_force` line
  and any failed rebuild, an edit dialog per key for the superuser, workspaces and subscriptions with their six verbs,
  and history.

## Acceptance

- `web:unit-test`, `web:lint`, `web:typecheck`, `web:story-render`, and `web:client-drift` pass.
- `blizzard:ui` over editing a budget and reading `in_force` move after a step, a failed rebuild shown on the `in_force`
  line, and a guest seeing no edit control.
- `web:manual-panel`.
