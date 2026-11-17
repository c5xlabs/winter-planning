---
name: new-item-form
external: github:paul-gross/blizzard#2277
depends-on:
  - form-chrome
code:
  - blizzard/web/projects/hub/src/app/board/new-hub-item/new-hub-item.ts::NewHubItem
  - blizzard/web/projects/hub/src/app/board/new-hub-item/new-hub-item-view.ts::NewHubItemView
---

# Migrate new-item entry

Moves the board's new-item form onto [forms.md](../spec/forms.md#system)'s shared path for `product:full-adoption`,
`product:immediate-validation`, and `product:error-at-field`.

## Change

- Add the typed title/body/project/urgency draft and schema with the existing lens-derived defaults and permissions.
- Move saving/invalidation and refusal mapping into a domain service; keep the presentational view field-tree-only.
- Remove its local pending/error/draft/layout implementation without adding a different creation capability.

## Acceptance

- Tests show missing title/body/project/urgency produces the appropriate field error and no request.
- Existing lens/default-urgency behavior and the returned created-item navigation are preserved.
- Success settles existing backlog/chunk refreshes; refusal retains typed content.
- `web:typecheck`, `web:unit-test`, and the existing surface's browser layout checks pass.
