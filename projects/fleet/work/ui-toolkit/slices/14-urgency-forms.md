---
name: urgency-forms
depends-on:
  - config-dialog-seam
code:
  - blizzard/web/projects/hub/src/app/admin/urgencies/urgencies-page.ts::UrgenciesPage
  - blizzard/web/projects/hub/src/app/admin/urgencies/urgencies.model.ts::URGENCY_FIELDS
---

# Migrate urgency configuration forms

Moves urgency create/edit and its read contract onto [forms.md](../spec/forms.md#system)'s typed path for
`product:full-adoption` and `product:domain-boundary`.

## Change

- Move slug, name, color, and mark rules out of generic field metadata; retain existing image/text behavior.
- Use the record's edit baseline and preserve separate ordering, default-setting, and lifecycle operations.
- Introduce the urgency domain projection for all current consumers, including proposal acceptance's existing view.

## Acceptance

- Tests cover required fields, mark read/format failure, cancel/pending behavior, and baseline-preserving sparse edits.
- Default-setting and ordering retain their existing mutation semantics and invalidations.
- `web:typecheck` and `web:unit-test` pass after all urgency consumers accept the domain contract; moved units leave
  boundary exemptions.
