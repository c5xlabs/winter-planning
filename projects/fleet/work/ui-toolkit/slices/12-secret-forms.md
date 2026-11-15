---
name: secret-forms
depends-on:
  - config-dialog-seam
code:
  - blizzard/web/projects/hub/src/app/admin/secrets/secrets-page.ts::SecretsPage
---

# Migrate secret configuration forms

Moves secret create/value replacement and feature readers to [forms.md](../spec/forms.md#system)'s typed path for
`product:full-adoption`, `product:error-at-field`, and `product:domain-boundary`.

## Change

- Keep secret values in the working draft, not a read-derived display model; preserve existing masked presentation.
- Use the captured revision for value replacement and retain unversioned lifecycle operations as unversioned.
- Route field and root refusals through shared submission instead of a dialog-specific error line.

## Acceptance

- Tests distinguish create, value replacement, and lifecycle calls, including which calls do and do not send If-Match.
- Required values block submission, refusal retains the typed value, and success closes only after invalidation settles.
- `web:typecheck`, `web:unit-test`, and the boundary gate pass with migrated secret units removed from exemptions.
