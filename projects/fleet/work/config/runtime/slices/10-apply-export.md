---
name: apply-export
depends-on:
  - settings-record
  - hosted-runners
  - host-records
code:
  - blizzard/src/blizzard/runner/host_config.py::decode_host_text
---

# Apply and export

Adds the runner's declarative apply and its export, emitted through the shared TOML emitter
([runner-api.md](../spec/runner-api.md) §Apply, §CLI). Delivers `product:runner-settings-document`.

## Change

- `POST /api/host/settings/apply` and `GET /api/host/settings/export`, and the `settings apply` and `settings export`
  verbs.
- Context: `bzh:config-apply` extends to the runner's document (`engineering:context-rules`).

## Acceptance

- `blizzard:unit-test` over the plan never retiring an omitted record and refusing an unknown runner.
- `blizzard:component-test` over a refused entry rolling back the whole document, a dry run reaching a validation
  refusal, and an export applied back writing nothing.
- `blizzard:cli-contract`, `web:client-drift`, and `blizzard:compat` pass.
