---
name: credential-forms
depends-on:
  - config-dialog-seam
code:
  - blizzard/web/projects/hub/src/app/admin/credentials/credentials-page.ts::CredentialsPage
  - blizzard/web/projects/hub/src/app/admin/credentials/credentials.model.ts::credentialFields
---

# Migrate credential configuration forms

Moves the credential forms added since the survey onto [forms.md](../spec/forms.md#system)'s typed path, serving
`product:full-adoption` and `product:domain-boundary` without exempting a later record type.

## Change

- Separate binding-derived draft rules from presentation field metadata; translate credential documents and binding
  data.
- Preserve dependent fields, existing secret handling, permissions, and sparse versioned edits.

## Acceptance

- Tests exercise more than one binding, required/hidden fields, dependent-choice changes, and correct omission of
  irrelevant fields in create/patch requests.
- Stale edits retain baseline/draft; hub refusals map through the shared outcome.
- `web:typecheck`, `web:unit-test`, and the boundary gate pass with moved credential units removed from exemptions.
