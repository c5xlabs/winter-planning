---
name: ui-job-move
depends-on:
  - board-build
  - epic:test-suite/fleet-tier
---

# The UI job's move

Moves the UI job onto the shared build, once its runtime fits inside the slowest `python-tests` shard. Delivers
`engineering:shared-board-build`; builds `spec/pr-gate.md` §The UI job. Shaped by
`decision:test-suite/optimization/ui-job-in-gate`.

## Change

- `gate.yml` gains `ui`, needing `frontend`, running on `web-dist`.
- The job leaves `upper-tiers.yml`.
- The required check is renamed `gate / ui` in branch protection and in blizzard-context's packaging page.
- `test_winter_free_ci.py` expects `("gate.yml", "ui")`.

## Acceptance

- The `ui` job passes on a pull request without building the board itself.
- `test_winter_free_ci.py` passes.
