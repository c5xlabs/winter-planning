---
name: cli-runner-local
depends-on:
  - cli-worker
  - runner-api-retirement
code:
  - blizzard/src/blizzard/runner/cli/runtime.py::read_stores
---

# Runner local-runtime command containers

Delivers the remaining runner command boundary in spec [Entrypoint resolution](../spec/entrypoints.md), using
[Scope ownership](../spec/scopes.md) and serving `engineering:selective-command-imports`, `engineering:container-only`,
and `engineering:ordered-teardown`.

## Change

- Migrate local host/tick/ingest/store/migration/check/offline-read commands to their selected area roots.
- Use adopted host/runner process builders and operation scopes; preserve configured binding before offline
  reads/writes.
- Keep local-provider imports inside selected command modules, preserve process/harness carrier inputs, and remove the
  remaining command/manual-store constructor paths.

## Acceptance

- Relevant CLI tests exercise graph selection, invoker overrides, offline binding and success/failure ownership.
- Runtime-consumer adaptations already landed before aggregate retirement remain in force; command migration does not
  recreate the retired service bags.
- Host/standalone tick paths share the intended process graph; borrowed runtimes remain open and owned ones close once.
- Fresh-interpreter tests keep local providers out of worker verbs; all command graphs, strict typing and CLI/config/
  supported-release contracts pass unchanged.
