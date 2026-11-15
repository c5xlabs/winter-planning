---
name: config-dialog-seam
depends-on:
  - form-chrome
code:
  - blizzard/web/projects/hub/src/app/admin/config-form-dialog.ts::ConfigFormDialog
  - blizzard/web/projects/hub/src/app/admin/config-edit.model.ts::ConfigFieldDef
---

# Introduce the typed config-dialog seam

Builds [forms.md](../spec/forms.md#interfaces)'s generic field-tree dialog under
`decision:ui-toolkit/incremental-replacement`, preparing `product:full-adoption`.

## Change

- Add the domain-field-tree entry whose field metadata controls presentation only.
- Preserve the old entry for unmoved callers; no caller is forced onto half-migrated validation.
- Preserve field differences, conditional visibility, dependent choices, and mark-input presentation.

## Acceptance

- Component tests render distinct typed field trees without metadata-based required/number checks or dialog-owned
  drafts.
- The old entry's existing caller tests still pass; the new entry's fixture exercises save, cancel, conditional fields,
  a file-read failure, and field/root refusals through shared form state.
- `web:typecheck` and `web:unit-test` pass with both entries; the transition adds no wire or user-visible redesign.
