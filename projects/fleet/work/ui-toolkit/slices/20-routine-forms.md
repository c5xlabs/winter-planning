---
name: routine-forms
depends-on:
  - form-chrome
code:
  - blizzard/web/projects/hub/src/app/garden/routines/routine-create-form.ts::RoutineCreateForm
  - blizzard/web/projects/hub/src/app/garden/routines/routine-panel.ts::FleetRoutinePanel
  - blizzard/web/projects/hub/src/app/garden/routines/gardening-routines-page.ts::GardeningRoutinesPage
---

# Migrate routine creation and inline edits

Moves routine input surfaces and read models to [forms.md](../spec/forms.md#system)'s typed path for
`product:full-adoption` and `product:domain-boundary`.

## Change

- Give creation and each existing inline edit their domain drafts, schema, translators, and service calls.
- Preserve graph/scope text entry and existing urgency choices; add no new picker or mutation semantics.
- Replace view-owned field state/layout while retaining lifecycle and run actions.

## Acceptance

- Tests pin create and each existing edit request, including empty/default handling and unchanged fields.
- Every versioned operation retains the original baseline; unversioned operations do not acquire If-Match.
- Component/browser tests cover local validation, hub refusal focus, draft retention, and narrow layout.
- `web:typecheck`, `web:unit-test`, and the boundary gate pass with migrated routine units removed from exemptions.
