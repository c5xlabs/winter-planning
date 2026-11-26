---
name: proposal-acceptance-assembly
external: github:paul-gross/blizzard#2295
depends-on:
  - retire-boundary-compatibility
  - aria-tabs
  - aria-chips-toolbars
  - aria-select-lists
  - aria-accordions
  - zone-combobox
  - garden-grids
code:
  - blizzard/web/projects/hub/src/app/garden/proposals/gardening-proposal-accept-dialog.ts::GardeningProposalAcceptDialog
  - blizzard/web/projects/hub/src/app/garden/proposals/gardening-proposal-accept-dialog-view.ts::GardeningProposalAcceptDialogView
  - blizzard/web/projects/hub/src/app/garden/proposals/garden-proposal.mutations.ts::injectAcceptGardenProposalMutation
---

# Assemble proposal acceptance from the completed toolkit

Delivers `product:assembled-surface` and the last form in `product:full-adoption` using
[forms.md](../spec/forms.md#interfaces)'s acceptance contract, under `decision:ui-toolkit/assembled-surface`.

## Change

- Replace proposal acceptance's custom draft, gating, layout, and refusal state with its domain/schema/service and kit
  parts.
- Keep mint as default, one reason across modes, one-time body prefill, and existing optional-field/request semantics.
- Add the shipped composition to the checked web exemplars while retaining cadence's versioned-edit example.
- Remove the superseded acceptance form helpers; add no invitation form or other unrelated capability.

## Acceptance

- Tests cover mint/decline switching, required trimmed decline reason, optional omitted fields, disabled hidden fields,
  tenant-default urgency, and preservation of edits after an input/query refresh.
- Field failures focus their controls; raced closure and network failures stay on the form and preserve the draft. No
  If-Match or automatic retry is invented; success waits for proposal-list/detail/count invalidation.
- A real-board browser case exercises both modes and refusal recovery; the view owns no draft/validation/error/layout
  implementation. `web:story-render` and `blizzard:exemplar-check` pass for the reused parts and checked examples.
- The final adoption census finds every live input surface on the shared path, no superseded helpers, and an empty
  boundary exemption list; the members grid, the cadence and routine select lists, and the zone picker satisfy their
  keyboard cases.
- `web:lint`, `web:typecheck`, `web:unit-test`, `web:structural-gate`, `web:shell-sweep`, and `web:bundle-composition`
  pass.
