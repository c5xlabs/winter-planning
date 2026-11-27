---
name: runner-host-domains
external: github:paul-gross/blizzard#2290
depends-on:
  - shared-chunk-domains
code:
  - blizzard/web/projects/hub/src/app/runners/fleet-page.ts::FleetPage
  - blizzard/web/projects/runner/src/app/runners/machine-page.ts::MachinePage
  - blizzard/web/projects/runner/src/app/host/host-page.ts::HostPage
---

# Move runner and host presentation onto domains

Migrates the hub's fleet/runner views and the runner panel's board, host, runner, ask, and event readers through
[boundary.md](../spec/boundary.md#system), serving `product:domain-boundary`.

## Change

- Translate each daemon's wire shapes at its own boundary and share only the domain concepts both apps actually use.
- Preserve identity/refusal/health judgments, host actions, lease state, pause controls, and local-versus-hub targeting.
- Keep query/cache, stream invalidation, permissions, and per-row pending behavior intact.

## Acceptance

- Domain/translator tests cover unknown states and identity/health refusals without re-deriving server judgments.
- Both apps' existing runner/host action and target-selection tests remain green.
- `web:typecheck`, `web:unit-test`, and board/panel browser checks pass; moved units and fixtures leave exemptions.
