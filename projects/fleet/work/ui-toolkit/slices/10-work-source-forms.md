---
name: work-source-forms
depends-on:
  - config-dialog-seam
code:
  - blizzard/web/projects/hub/src/app/admin/work-sources/work-sources-page.ts::WorkSourcesPage
---

# Migrate work-source configuration forms

Moves work-source create/edit and read translation to [forms.md](../spec/forms.md#system)'s shared path for
`product:full-adoption`, `product:immediate-validation`, and `product:domain-boundary`, under
`decision:ui-toolkit/forge-required`.

## Change

- Add typed drafts, local rules, request/refusal translation, and a service over existing mutations.
- Preserve filtered credential/forge choices and permissions; require forge in the form, not by a wire change.
- Move existing work-source feature readers and builders onto domain types.

## Acceptance

- Tests pin required forge, dependent-choice clearing, existing defaults, and sparse edit requests.
- Field refusals focus the field; stale edits retain draft and original revision without retry.
- `web:typecheck`, `web:unit-test`, and `web:structural-gate` pass; migrated units leave exemptions.
