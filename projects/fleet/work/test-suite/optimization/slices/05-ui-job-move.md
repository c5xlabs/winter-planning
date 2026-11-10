---
name: ui-job-move
depends-on:
  - board-build
  - epic:test-suite/fleet-tier
code:
  - blizzard/.github/workflows/upper-tiers.yml::ui
  - blizzard/tests/guards/test_winter_free_ci.py::test_the_scan_sees_the_test_running_jobs
---

# The UI job's move

Moves the UI job onto the shared build, once its runtime fits inside the slowest `python-tests` shard. Delivers
`engineering:shared-board-build`; builds `spec/pr-gate.md` §The UI job and §The required check. Shaped by
`decision:test-suite/optimization/ui-job-in-gate`; waits on the open questions in `spec/pr-gate.md` of which mock commit
it proves against and which callers run it.

## Change

- `gate.yml` gains `ui`, needing `frontend`, running `mise run playwright -- --project='ui*'` on `web-dist`.
- The job leaves `upper-tiers.yml`.
- The required check is renamed `gate / ui` in branch protection and in blizzard-context's packaging page.
- `test_winter_free_ci.py`'s set of test-running jobs gains `("gate.yml", "ui")`.

## Acceptance

- The `ui` job passes on a pull request without building the board itself.
- `test_winter_free_ci.py` passes.
