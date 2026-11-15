---
name: proposal-pass-form
depends-on:
  - form-chrome
code:
  - blizzard/web/projects/hub/src/app/garden/proposals/gardening-proposal-pass-dialog.ts::GardeningProposalPassDialog
  - blizzard/web/projects/hub/src/app/garden/proposals/garden-proposal.mutations.ts::injectPassGardenProposalMutation
---

# Migrate proposal pass

Moves proposal pass to [forms.md](../spec/forms.md#system)'s domain/form/service path for `product:full-adoption` and
`product:error-at-field`. Proposal acceptance remains on its existing form until the capstone
(`decision:ui-toolkit/assembled-surface`).

## Change

- Declare the reason rule in the domain form schema and route submission through shared kit parts and outcomes.
- Keep the existing terminal closure semantics and proposal-list/detail/count invalidations.
- Translate shared proposal read data without rebuilding the acceptance form early.

## Acceptance

- Blank required reason blocks Save with a field error; a hub field refusal focuses reason.
- A raced closure's 409 remains a form-level refusal, retains the draft, and does not retry.
- Tests confirm success waits for the existing invalidations; the still-unmoved acceptance form remains functional.
- `web:typecheck`, `web:unit-test`, and browser checks of refusal/pending states pass.
