---
name: form-save-contracts
external: github:paul-gross/blizzard#2257
depends-on:
  - domain-boundary-gate
code:
  - blizzard/web/scripts/placement-sweep.js::sweepPlacement
---

# Add the shared form-save contracts

Builds [forms.md](../spec/forms.md#domain) and [its interfaces](../spec/forms.md#interfaces): `SaveOutcome`, refusal
translation, `EditBaseline`, numeric parsing, and submission helpers. Serves `product:error-at-field`,
`product:no-silent-reset`, and `product:immediate-validation`.

## Change

- Add the shared contracts without moving an application form yet.
- Add only the documented `core/forms` bootstrap placement exemption from [forms.md](../spec/forms.md#system), under
  `decision:ui-toolkit/shared-form-bootstrap`.
- Normalize refusal paths and kinds once, preserving a form-level fallback.
- Standardize captured revision baselines and lossless whole-number parsing; record the edit lifecycle in
  blizzard-context's mutation guidance.

## Acceptance

- Pure tests cover nested/indexed refusal paths, leading location segments, kind-only refusals, and unknown/network
  failures without Angular `TestBed`.
- Numeric tests cover zero, leading zeros, the safe-integer limit, unsafe integers, overflow, blank, and malformed
  input; an invalid parse cannot become a transmitted default.
- Submission tests show invalid forms send nothing, all fields become touched, hub errors target the right field/root,
  and the first invalid control receives focus.
- `web:typecheck` compiles the shared generic contracts and `web:unit-test` passes.
- `web:structural-gate` passes before app adoption; detector fixtures show the named bootstrap unit passes while an
  unrelated unused shared unit still fails.
