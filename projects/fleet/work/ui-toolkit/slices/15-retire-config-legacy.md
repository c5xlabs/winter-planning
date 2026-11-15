---
name: retire-config-legacy
depends-on:
  - project-forms
  - repository-forms
  - work-source-forms
  - forge-forms
  - secret-forms
  - credential-forms
  - urgency-forms
code:
  - blizzard/web/projects/hub/src/app/admin/config-form-dialog.ts::ConfigFormDialog
  - blizzard/web/projects/hub/src/app/admin/config-edit.model.ts::missingRequired
  - blizzard/web/projects/hub/src/app/admin/config-edit.model.ts::invalidNumbers
---

# Retire the legacy config-form path

Closes the config-dialog transition in [forms.md](../spec/forms.md#interfaces) under
`decision:ui-toolkit/incremental-replacement`, delivering the admin cleanup in `product:full-adoption`.

## Change

- Remove the old dialog entry, local draft helpers, metadata-based required/number checks, and text-number conversion.
- Keep field metadata only for presentation; preserve still-used field-difference presentation.

## Acceptance

- A source census finds no legacy dialog caller, `missingRequired`, `invalidNumbers`, or generic text-number conversion.
- Tests cover the typed dialog across all seven current record types, including conditional and numeric fields.
- `web:typecheck`, `web:unit-test`, and `web:structural-gate` pass with no migration-only admin compatibility branch.
