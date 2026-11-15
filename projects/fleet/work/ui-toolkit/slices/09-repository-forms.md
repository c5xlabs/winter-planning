---
name: repository-forms
depends-on:
  - config-dialog-seam
code:
  - blizzard/web/projects/hub/src/app/admin/repositories/repositories-page.ts::RepositoriesPage
  - blizzard/web/projects/hub/src/app/admin/repositories/repositories.model.ts::REPOSITORY_FIELDS
---

# Migrate repository configuration forms

Delivers [forms.md](../spec/forms.md#domain)'s repository draft and numeric/default contract for
`product:no-silent-reset`, `product:immediate-validation`, and `product:full-adoption`, under
`decision:ui-toolkit/forge-required` and `decision:ui-toolkit/explicit-default`.

## Change

- Move create/edit, validation, sparse translation, and refusal routing into the repository domain/form/service.
- Require forge locally without changing its optional wire field; expose the explicit park-timeout default choice.
- Preserve the required forge-filtered credential, optional name/landing policy, and Allow no CI checkbox from the
  reconciled draft/translation contract; do not revive the retired token-secret field.
- Translate repository reads and migrate the feature's views and builders.

## Acceptance

- A missing forge is a marked field error; slug/coordinate refusals map to their specified fields.
- Tests require credential, clear an incompatible choice on forge change, transmit both true and false `allow_no_ci`,
  and preserve create omissions and sparse edits for all existing fields.
- Blank, malformed, unsafe, and overflowing stated timeouts send nothing; explicit default alone sends null.
- Service tests repeat those conversion checks without a mounted form and retain the baseline revision on edit.
- `web:typecheck`, `web:unit-test`, and the boundary gate pass with moved units removed from exemptions.
