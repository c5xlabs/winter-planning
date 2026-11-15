---
name: cli-hub-local
depends-on:
  - cli-hub-client
  - hub-legacy-retirement
---

# Hub local-runtime command containers

Delivers the hub local-verb boundary in spec [Entrypoint resolution](../spec/entrypoints.md), using
[Scope ownership](../spec/scopes.md) and serving `engineering:selective-command-imports`, `engineering:container-only`,
and `engineering:ordered-teardown`.

## Change

- Migrate the complementary local/server/store/migration/check commands under `hub/cli/` to their selected area roots.
- Use the hub process graph for hosted startup, with ownership handed to the app; keep one-shot store resources owned by
  the command and externally supplied inputs borrowed.
- Remove the hub CLI's remaining manual collaborator construction and transition bridges.

## Acceptance

- CLI tests exercise each local verb's selected graph, carrier inputs and failure cleanup.
- Starting a hosted graph does not build/dispose duplicate process resources; one-shot exit closes its resources.
- Fresh-interpreter tests preserve the separation from client-only verbs; strict typing, relevant CLI tests and command/
  config/compatibility contracts pass unchanged.
