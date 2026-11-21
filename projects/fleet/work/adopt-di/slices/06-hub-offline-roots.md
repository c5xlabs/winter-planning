---
name: hub-offline-roots
depends-on:
  - hub-configured-adapters
code:
  - blizzard/src/blizzard/hub/app.py::rotate_secret_keys
  - blizzard/src/blizzard/hub/app.py::import_legacy_config
---

# Offline hub roots as compatibility entrypoints

Narrows the offline hub roots named in spec [Mixed ownership](../spec/migration.md#area-ownership) ahead of the
aggregates they read, under `decision:adopt-di/declared-cutover`. Their commands migrate with the hub's local CLI verbs.

## Change

- Turn `rotate_secret_keys` and `import_legacy_config` into bounded compatibility entrypoints: same call signatures,
  named capabilities resolved from a hub root the call owns ([Scope ownership](../spec/scopes.md#interfaces)), no
  `HubCore` or `TenantStores` field read.
- `rotate_secret_keys` walks open tenants through `TenantContainers.every_open` and the registered `secret_cipher`;
  `import_legacy_config` takes the registered config authoring in place of `build_config_authoring`.
- `check_forges` opens its own store and reads forge addresses over a raw connection, reading no aggregate, and is left
  unchanged.

## Acceptance

- `secret rotate-key` and `config import-legacy` behave unchanged against a real store: rotated values decrypt with the
  new key in every tenant, and an imported config matches today's, with unchanged output and exit status
  (`blizzard:component-test`); the command surface is unchanged (`blizzard:cli-contract`).
- Neither entrypoint constructs `HubCore` or reads a `TenantStores` field; its root closes on success and on failure.
- `blizzard:typecheck` and the graph/ownership gates pass.
