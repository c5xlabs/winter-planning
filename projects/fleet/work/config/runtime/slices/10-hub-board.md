---
name: hub-board
depends-on:
  - hub-settings
---

# The Hub settings page

Adds the Hub settings page to the hub's board ([hub.md](../spec/hub.md) §The board). Delivers the board half of
`product:hub-settings-record`.

## Change

- The Hub settings page, shown only to a session holding `SETTINGS_ADMIN`.

## Acceptance

- `web:unit-test`, `web:lint`, `web:typecheck`, `web:story-render`, and `web:client-drift` pass.
- `blizzard:ui` over editing a sweep interval as a hub administrator, and a tenant superuser finding no page.
