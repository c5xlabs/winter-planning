---
name: scope-forms
external: github:paul-gross/blizzard#2275
depends-on:
  - form-chrome
code:
  - blizzard/web/projects/hub/src/app/garden/scopes/gardening-scopes-page.ts::GardeningScopesPage
  - blizzard/web/projects/hub/src/app/garden/scopes/scope-panel.ts::FleetScopePanel
---

# Migrate scope input surfaces

Moves scope creation and inline edits to [forms.md](../spec/forms.md#system)'s typed path, serving
`product:full-adoption`, `product:error-at-field`, and `product:domain-boundary`.

## Change

- Put existing scope fields and rules in domain drafts/schemas; use services over the existing writes.
- Translate scope reads and migrate the feature's models/builders without altering links or lifecycle actions.
- Remove local form draft/layout/refusal helpers.

## Acceptance

- Tests exercise every existing scope input operation, required/optional fields, and unchanged request semantics.
- Refusals retain the draft, show at the correct level, and focus the first invalid field.
- `web:typecheck`, `web:unit-test`, and browser validation/layout checks pass; moved scope units leave exemptions.
