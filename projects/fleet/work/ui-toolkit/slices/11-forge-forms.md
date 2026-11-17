---
name: forge-forms
external: github:paul-gross/blizzard#2265
depends-on:
  - config-dialog-seam
code:
  - blizzard/web/projects/hub/src/app/admin/forges/forges-page.ts::ForgesPage
---

# Migrate forge configuration forms

Moves forge create/edit to [forms.md](../spec/forms.md#system)'s typed path for `product:full-adoption`,
`product:error-at-field`, and `product:domain-boundary`.

## Change

- Add forge domain, schema, translators, and service; use the typed dialog instead of the legacy entry.
- Preserve binding-dependent fields and existing lifecycle/read behavior while translating generated types at the
  boundary.

## Acceptance

- Tests cover conditional fields, required markers, sparse edits, and the forge's existing revision-conflict refusal.
- Field/root failures retain the draft and settle the existing mutation invalidations.
- `web:typecheck`, `web:unit-test`, and the boundary gate pass with moved forge units removed from exemptions.
