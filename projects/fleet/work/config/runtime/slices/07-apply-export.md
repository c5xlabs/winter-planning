---
name: apply-export
depends-on:
  - hosted-runners
  - host-records
---

# Apply and export

Adds the runner's declarative apply and its export, and the shared TOML emitter ([runner-api.md](../spec/runner-api.md)
§Apply). Delivers `product:runner-settings-document`.

## Change

- The runner's apply and export routes and their `settings apply` and `settings export` verbs.
- `foundation/toml_emit.py`.
- Context: `bzh:config-apply` extends its scope to the runner's document (`engineering:context-rules`).

## Acceptance

- `blizzard:unit-test` over the plan never retiring an omitted record and refusing an unknown runner.
- `blizzard:component-test` over a refused entry rolling back the whole document, a dry run reaching a validation
  refusal, and an export applied back writing nothing.
- `blizzard:cli-contract`, `web:client-drift`, and `blizzard:compat` pass.
