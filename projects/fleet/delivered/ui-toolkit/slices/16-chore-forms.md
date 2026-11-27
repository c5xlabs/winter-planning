---
name: chore-forms
external: github:paul-gross/blizzard#2270
depends-on:
  - cadence-form
code:
  - blizzard/web/projects/hub/src/app/garden/cadences/chore-form.ts::FleetChoreForm
  - blizzard/web/projects/hub/src/app/garden/cadences/cadence-panel.ts::FleetCadencePanel
---

# Migrate cadence chore forms

Moves chore add/edit to [forms.md](../spec/forms.md#system)'s domain/form/service path, delivering this part of
`product:full-adoption` and preserving [the captured edit flow](../spec/forms.md#flows).

## Change

- Move chore draft state and rules out of the view; keep existing routine, scope, urgency, and note semantics.
- Translate an add/edit/remove into the existing whole-chores cadence patch against its captured cadence revision.
- Remove chore-specific field layout and save helpers without altering cadence lifecycle actions.

## Acceptance

- Tests show add/edit/remove preserve untouched chores and stable identities in the submitted list.
- A concurrent cadence edit yields a form-level refusal without rebuilding the patch against a refreshed list.
- Required/rule errors focus their fields; refusals retain the chore draft.
- `web:typecheck`, `web:unit-test`, and the boundary gate pass; browser checks cover form errors and narrow layout.
