---
name: delivery-domains
external: github:paul-gross/blizzard#2288
depends-on:
  - shared-chunk-domains
code:
  - blizzard/web/projects/hub/src/app/delivery/delivery-page.ts::DeliveryPage
  - blizzard/web/projects/hub/src/app/delivery/delivery-chunk-page.ts::DeliveryChunkPage
---

# Move delivery read models onto domains

Migrates delivery lists/history/detail through [boundary.md](../spec/boundary.md#system), serving
`product:domain-boundary` without changing delivery behavior.

## Change

- Add delivery domain/read translation and move remaining generated presentation types and builders behind it.
- Keep links, failure labels, history ordering, permissions, and existing shared chunk presentation unchanged.

## Acceptance

- Pure tests cover success/failure/history and unknown-value rendering without reconstructing backend decisions.
- Existing delivery interaction and layout checks pass with domain fixtures.
- `web:typecheck`, `web:unit-test`, and the boundary gate pass with migrated delivery units removed from exemptions.
