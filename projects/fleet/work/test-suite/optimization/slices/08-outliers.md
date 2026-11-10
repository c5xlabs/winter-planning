---
name: outliers
depends-on:
  - durations-recorder
code:
  - blizzard/.github/workflows/gate.yml::python-tests
  - blizzard/tests/component/runner/harness/opencode/compatibility/test_diagnostic.py::_probe
---

# The outliers

Speeds the slow outliers, one change per file the measurement names, then records the gate against the baseline.
Delivers `engineering:outlier-triage` and `engineering:gate-measured`; builds `spec/outliers.md`. Waits on the open
question in `spec/outliers.md` of where `test_diagnostic.py`'s cost lies, which the measurement answers.

## Change

- `gate.yml`'s `workflow_dispatch` input adding `--durations=0`; the slowest fifty items of one run recorded beside the
  baseline.
- Per outlier, the slow part kept, shrunk, or replaced, the choice recorded in the commit message.
- Fixtures built identically per test, such as `fake-opencode`, shared at the widest scope no test mutates.
- Each change's description records the job it shortened, before and after.
- Once the split, the amd64-only image, and the outliers have landed, the baseline is taken again with the durations
  recorder.

## Acceptance

- Every changed test asserts what it asserted before (`engineering:assertions-unchanged`).
- The gate's critical path is recorded against the baseline of 18 October 2026.
