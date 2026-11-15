---
name: board-domains
depends-on:
  - shared-chunk-domains
code:
  - blizzard/web/projects/hub/src/app/board/board-page.ts::BoardPage
---

# Move hub board read models onto domains

Migrates the hub board's remaining chunks, tracks, activity, and detail read models through
[boundary.md](../spec/boundary.md#interfaces), serving `product:domain-boundary`.

## Change

- Translate board queries and replace generated inputs in page, card, track, and detail models.
- Keep lensed reads, navigation, pending overrides, backend judgments, and query/cache identities unchanged.
- Reuse the already-migrated shared chunk contracts and form services.

## Acceptance

- Model/translator tests pin populated, loading, empty, failed, and unknown-status behavior with domain builders.
- Existing board navigation, pending action, timeline, and lensed-read tests remain green.
- `web:typecheck`, `web:unit-test`, and affected browser layout cases pass; moved board units leave boundary exemptions.
