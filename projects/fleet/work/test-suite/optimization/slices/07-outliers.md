---
name: outliers
depends-on:
  - durations-recorder
---

# The outliers

Speeds the slow outliers, one change per file the measurement names. Delivers `engineering:outlier-triage`; builds
`spec/outliers.md`.

## Change

- The measurement, through `gate.yml`'s `workflow_dispatch` input, recorded beside the baseline.
- Per outlier, the slow part kept, shrunk, or replaced, the choice recorded in the commit message.
- Fixtures built identically per test, such as `fake-opencode`, shared at the widest scope no test mutates.
- Each change's description records the job it shortened, before and after (`engineering:gate-measured`).

## Acceptance

- Every changed test asserts what it asserted before (`engineering:assertions-unchanged`).
