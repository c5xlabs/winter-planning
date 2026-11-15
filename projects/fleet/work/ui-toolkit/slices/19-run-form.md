---
name: run-form
depends-on:
  - form-chrome
code:
  - blizzard/web/projects/hub/src/app/garden/runs/gardening-run-dialog.ts::GardeningRunDialog
---

# Migrate the run dialog

Moves the run dialog to [forms.md](../spec/forms.md#system)'s domain/form/service path for `product:full-adoption`,
`product:immediate-validation`, and `product:domain-boundary`.

## Change

- Bind the existing run choices and typed fields to a schema-backed draft owned by the container.
- Retain current routine/scope/mode/urgency semantics and native-radio presentation through the option-group control.
- Keep one saving service over the existing mutation; remove local draft and refusal plumbing.

## Acceptance

- Schema/translator tests cover each existing mode and its conditional fields, defaults, and omitted values.
- Component tests show invalid conditional fields block only when applicable and required markers follow the schema.
- Pending, refusal, and success cases preserve current cancellation/invalidation behavior.
- `web:typecheck`, `web:unit-test`, and browser keyboard/error checks pass; moved run units leave boundary exemptions.
