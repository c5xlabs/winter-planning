---
name: shared-core-domains
depends-on:
  - domain-boundary-gate
code:
  - blizzard/web/projects/fleet/src/lib/core/chunk-lanes.ts::STATUS_LANE
  - blizzard/web/projects/fleet/src/lib/sse/fleet-live.ts::HubEventPayloads
---

# Move shared core and transport consumers onto domains

Builds [boundary.md](../spec/boundary.md#domain)'s shared domain/translator layer for core vocabularies, host facts, and
shared read/stream consumers, serving `product:domain-boundary`.

## Change

- Put shared concepts in `fleet`'s kernel, translate generated values once, and preserve explicit unknown handling.
- Preserve backend-supplied judgments, event/stream semantics, query caches, and invalidation keys.
- Give still-unmoved readers only the temporary data-access projections permitted by
  `decision:ui-toolkit/incremental-replacement`.

## Acceptance

- Pure tests translate known and unforeseen vocabularies and preserve the backend's terminal/other supplied judgments.
- Existing stream and cache-invalidation tests still pass; no component-side translator or second cache is introduced.
- Core/model specs and stories build domain literals; focused boundary specs retain wire fixtures in their permitted
  home.
- `web:typecheck`, `web:unit-test`, and `web:structural-gate` pass with migrated shared-core units removed from
  exemptions.
