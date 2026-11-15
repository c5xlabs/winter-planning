---
name: cadence-form
depends-on:
  - form-chrome
code:
  - blizzard/web/projects/hub/src/app/garden/cadences/cadence-form.ts::FleetCadenceForm
  - blizzard/web/projects/hub/src/app/garden/cadences/gardening-cadences-page.ts::GardeningCadencesPage
  - blizzard/web/projects/hub/src/app/garden/cadences/cadence-edit.mutations.ts::CadenceEditVars
---

# Migrate the cadence form end to end

Delivers the cadence example in [forms.md](../spec/forms.md#domain), [its save flow](../spec/forms.md#flows), and
[boundary.md](../spec/boundary.md#interfaces), serving `product:immediate-validation`, `product:error-at-field`,
`product:domain-boundary`, and `product:full-adoption`.

## Change

- Introduce cadence domain/read translation, schema, refusal map, and service; replace the form's local helpers.
- Capture edit identity, revision, and original draft together; retain the existing sparse mutation and invalidations.
- Add the versioned-edit exemplar and its domain builder; adapt existing cadence readers to the introduced contract.
- Keep the zone's current text control until the combobox adoption.

## Acceptance

- Domain/schema tests cover slug creation, cron/zone pairing, spacing/anchor coupling, and an empty cadence.
- Service tests send the original baseline revision after a query refresh; stale-edit 409s preserve baseline and draft,
  never retry, and show at form level. Success waits for existing invalidations.
- Component/browser cases cover required markers, inline failures, first-error focus, hub slug refusal, and retained
  draft.
- `web:typecheck`, `web:unit-test`, `web:structural-gate`, and `blizzard:exemplar-check` pass; moved units leave
  exemptions.
