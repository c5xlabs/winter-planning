# The pull request gate

## The board's build on every pull request

`gate.yml`'s `frontend` job gains a step after `npm ci`: `npm run build`, which runs `build:hub` and `build:runner` into
`src/blizzard/static/hub` and `src/blizzard/static/runner`, as `angular.json`'s `outputPath`s place them. A template
error the runner's board alone would catch now fails the pull request.

The job then uploads `src/blizzard/static/` as the artifact `web-dist`, kept for one day, which is the hand-off the UI
job reads. `bundle-check` keeps its own build, since it builds with `--stats-json` into a scratch directory.

## The UI job

`epic:test-suite/fleet-tier` lands the UI tests with a job of their own in `upper-tiers.yml`, under `full_suite`, that
builds the board itself, as the required check `upper-tiers / ui (blizzard:ui)`. `gate.yml` gains `ui`, needing
`frontend`, with the two-repo checkout of `blizzard` and `blizzard-mock` with `persist-credentials: false`, `uv sync` in
both, Node from `mise.toml`, `npm ci` in `tests/`, `npx playwright install --with-deps chromium`, then `web-dist`
downloaded into `src/blizzard/static/` and `mise run playwright -- --project='ui-*'`. A failure uploads the same
evidence the service shards do.

The move (`decision:test-suite/optimization/ui-job-in-gate`, `engineering:shared-board-build`) renames the required
check to `gate / ui` in branch protection and blizzard-context's packaging page, and `test_winter_free_ci.py`'s set of
expected test jobs moves its entry from `upper-tiers.yml` to `("gate.yml", "ui")`.
