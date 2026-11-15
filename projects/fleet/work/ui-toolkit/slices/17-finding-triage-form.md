---
name: finding-triage-form
depends-on:
  - form-chrome
code:
  - blizzard/web/projects/hub/src/app/garden/findings/gardening-finding-triage-dialog.ts::GardeningFindingTriageDialog
---

# Migrate finding triage

Rebuilds the finding-triage dialog using [forms.md](../spec/forms.md#system)'s shared path, serving
`product:full-adoption`, `product:error-at-field`, and `product:domain-boundary`.

## Change

- Put the existing triage fields and local rules in a domain draft/schema and save through a domain service.
- Translate finding reads/refusals at the boundary and remove view-owned draft/error/layout helpers.
- Preserve current state transitions and free-text fields; add no extra picker.

## Acceptance

- Schema and translator tests cover each existing triage choice and its required/omitted fields.
- Invalid input sends nothing; hub field/root failures retain the draft and focus the applicable field.
- Existing triage mutation/invalidation behavior is unchanged; `web:typecheck` and `web:unit-test` pass.
- Browser checks cover errors and pending state; moved finding units leave boundary exemptions.
