# The pull request gate

Paths are relative to the `blizzard` repo root.

## CI

### The board's build

```yaml
# .github/workflows/gate.yml — product:board-build-on-pr
frontend:
  name: eslint + vitest + client drift
  steps:
    # … checkout, Node 22, as today
    - name: npm ci
      run: cd web && npm ci
    # `npm run build` is build:hub then build:runner, into src/blizzard/static/hub and src/blizzard/static/runner
    # (angular.json's outputPaths). A template error either board alone catches fails here.
    - name: build both boards
      run: cd web && npm run build
    - name: hand the build to the ui job
      uses: actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02 # v4.6.2
      with:
        name: web-dist
        path: src/blizzard/static/
        retention-days: 1
    # … eslint, vitest, structural gate, chromium, story render, as today
    - name: bundle composition # keeps its own `ng build <app> --output-path <scratch> --stats-json` (Open questions)
      run: cd web && npm run bundle-check
    # … generated-client drift, as today
```

### The UI job

`epic:test-suite/fleet-tier` lands the job this moves: `upper-tiers.yml`'s `ui`, behind `full_suite`, which builds the
board itself through `mise run ui`'s `web-build` dependency, reported as `upper-tiers / ui (blizzard:ui)`.

```yaml
# .github/workflows/gate.yml — engineering:shared-board-build, decision:test-suite/optimization/ui-job-in-gate
ui:
  name: ui
  needs: [frontend]
  runs-on: ubuntu-latest
  steps:
    - name: checkout blizzard
      uses: actions/checkout@v4
      with:
        path: blizzard
        persist-credentials: false
    - name: resolve and checkout blizzard-mock
      uses: ./blizzard/.github/actions/resolve-mock-ref # no `sha:` from upper-tiers' mock-ref (Open questions)
    # … uv with both lockfiles, Node from mise.toml, git identity, uv sync in blizzard and blizzard-mock,
    #   npm ci in blizzard/tests, as upper-tiers.yml's ui today; no npm ci in web/
    - name: install Chromium for the ui projects (Playwright)
      run: cd blizzard/tests && npx playwright install --with-deps chromium
    - name: the board frontend built
      uses: actions/download-artifact@v4
      with:
        name: web-dist
        path: blizzard/src/blizzard/static/
    # `mise run playwright`, not `mise run ui`: the ui task's web-build dependency would build the board again
    - name: ui tier — a Chromium page against the board each test's hub serves
      timeout-minutes: 30
      env:
        CI: "1"
        MISE_TRUSTED_CONFIG_PATHS: ${{ github.workspace }}/blizzard
      run: cd blizzard && mise run playwright -- --project='ui*'
    - name: keep the failure evidence
      if: ${{ failure() }}
      uses: actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02 # v4.6.2
      with:
        name: ui
        path: |
          blizzard/tests/playwright-report/
          blizzard/tests/test-results/
          blizzard/tests/.run/
        retention-days: 14
        if-no-files-found: ignore
```

`upper-tiers.yml` loses its `ui` job.

### The required check

| Where                                                                                  | Before                           | After                      |
| -------------------------------------------------------------------------------------- | -------------------------------- | -------------------------- |
| Branch protection on `master`                                                          | `upper-tiers / ui (blizzard:ui)` | `gate / ui`                |
| `blizzard-context`'s `verification/blizzard/commands/packaging.md` required-check list | `upper-tiers / ui (blizzard:ui)` | `gate / ui`                |
| `tests/guards/test_winter_free_ci.py::test_the_scan_sees_the_test_running_jobs` set    | no `ui` entry                    | gains `("gate.yml", "ui")` |

## Open questions

- Whether `bundle-check` can read its stats from the `frontend` job's build rather than building its own. The epic takes
  the saving only if `bundle-check.js` can be pointed at it without changing what it checks. — engineering
- Which `blizzard-mock` commit `gate.yml`'s `ui` proves against. `upper-tiers.yml`'s `mock-ref` resolves one commit for
  every tier of a run and `push.yml` records it as `proven-against`; a job in `gate.yml` cannot read that output and
  would resolve its own. — engineering
- Whether `gate.yml`'s `ui` runs for every caller. Behind `full_suite` it ran on pull requests and releases only;
  `push.yml` also calls `gate.yml`. — engineering
