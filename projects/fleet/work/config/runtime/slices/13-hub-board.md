---
name: hub-board
external: github:paul-gross/blizzard#2137
depends-on:
  - hub-settings
code:
  - blizzard/web/projects/hub/src/app/admin/admin.routes.ts::ADMIN_ROUTES
  - blizzard/web/projects/hub/src/app/admin/config-record-panel.ts::ConfigRecordPanel
  - blizzard/web/projects/hub/src/app/admin/config-revisions.ts::ConfigRevisions
  - blizzard/web/projects/hub/src/app/admin/config-field-diff.ts::ConfigFieldDiff
---

# The Hub settings page

Adds the Hub settings page to the hub's board ([hub.md](../spec/hub.md) §The board). Delivers the board half of
`product:hub-settings-record`.

## Change

- The Hub settings page, shown only to a session holding `SETTINGS_ADMIN`, built from the admin shell's record panel,
  revisions, and field-diff components.

## Acceptance

- `web:unit-test`, `web:lint`, `web:typecheck`, `web:story-render`, and `web:client-drift` pass.
- `blizzard:ui` over editing a sweep interval as a hub administrator, and a tenant superuser finding no page.
