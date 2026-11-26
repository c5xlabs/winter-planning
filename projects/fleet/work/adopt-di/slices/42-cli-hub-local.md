---
name: cli-hub-local
external: github:paul-gross/blizzard#2496
depends-on:
  - cli-hub-client
  - hub-offline-roots
  - hub-legacy-retirement
code:
  - blizzard/src/blizzard/hub/cli/runtime.py::host
  - blizzard/src/blizzard/hub/cli/runtime.py::init
  - blizzard/src/blizzard/hub/cli/runtime.py::migrate_cmd
  - blizzard/src/blizzard/hub/cli/__init__.py::_COMMANDS
  - blizzard/src/blizzard/hub/cli/config.py::config_import_legacy
  - blizzard/src/blizzard/hub/cli/config.py::config_check_forges
  - blizzard/src/blizzard/hub/cli/secret.py::secret_rotate_key
---

# Hub local-runtime command containers

Delivers the hub local-verb boundary in spec [Entrypoint resolution](../spec/entrypoints.md), using
[Scope ownership](../spec/scopes.md) and serving `engineering:selective-command-imports`, `engineering:container-only`,
and `engineering:ordered-teardown`.

## Change

- Migrate the complementary local/server/store/migration/check commands under `hub/cli/` to their selected area roots.
  Each builds the hub's `PROCESS`/`TENANT` translation as a root the command owns and closes on exit
  ([Interfaces](../spec/scopes.md#interfaces)); that is the command's one container, its command-local services
  registered at the hub translation's `PROCESS` lifetime.
- Migrate the offline verbs that share groups with client verbs: `config import-legacy`, `config check-forges` and
  `secret rotate-key`, which call the compatibility entrypoints `hub-offline-roots` left; retire those entrypoints once
  these verbs resolve from their command root.
- Use the hub process graph for hosted startup, with ownership handed to the app; keep one-shot store resources owned by
  the command and externally supplied inputs borrowed.
- Remove the hub CLI's remaining manual collaborator construction and transition bridges.

## Acceptance

- `blizzard:component-test` (`tests/component/hub/cli`) tests exercise each local verb's selected graph, carrier inputs
  and failure cleanup.
- `blizzard:component-test`: starting a hosted graph does not build/dispose duplicate process resources; one-shot exit
  closes its resources.
- `blizzard:unit-test` fresh-interpreter loaded-module tests (`tests/support/loaded_modules.py`) preserve the separation
  from client-only verbs; `blizzard:typecheck`, `blizzard:cli-contract`, `blizzard:config-contract` and
  `blizzard:compat` pass unchanged.
