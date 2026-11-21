---
name: runner-api-settings
depends-on:
  - runner-api-services
code:
  - blizzard/src/blizzard/runner/api/settings.py::router
  - blizzard/src/blizzard/runner/api/host_settings.py::router
  - blizzard/src/blizzard/runner/api/workspace_prompt.py::router
---

# Settings and workspace runner routes

Migrates one route family of the runner-API row in spec [Mixed ownership](../spec/migration.md#area-ownership) onto the
resolution mechanism `runner-api-services` built, delivering `engineering:narrow-api-edge`'s route portion under
`decision:adopt-di/declared-cutover` and `decision:adopt-di/adoption-batch-size`.

## Change

- Move these route modules from `RunnerWiring` accessors to declared services: `runner/api/settings.py`,
  `runner/api/host_settings.py`, `runner/api/workspace_prompt.py`.
- Register the settings apply and export services (`epic:config/runtime`), after the host settings, the workspace and
  subscription records and the runner settings writers they drive, and narrow their routes to them.
- Narrow the shared helpers these modules use to typed arguments; a helper another family still calls keeps its old
  callers.

## Acceptance

- `blizzard:component-test`: each migrated route keeps today's refusals (`503` with its label, `404` unknown or drained
  runner, `503` halted runner) and responses; two-runner tests preserve isolation and shared host data.
- `blizzard:component-test`: apply and export behave as `epic:config/runtime` specifies: a refused entry rolls back the
  document, and an export applied back writes nothing.
- `blizzard:unit-test`: the every-route walker resolves every migrated route at its actual scope; no migrated module
  reads `RunnerWiring`.
- `blizzard:typecheck` and `blizzard:compat` pass with unchanged wire behavior.
