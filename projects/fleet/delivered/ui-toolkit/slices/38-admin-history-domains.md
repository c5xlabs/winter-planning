---
name: admin-history-domains
external: github:paul-gross/blizzard#2292
depends-on:
  - retire-config-legacy
code:
  - blizzard/web/projects/hub/src/app/admin/config-history.model.ts::revisionRows
  - blizzard/web/projects/hub/src/app/admin/changes/changes.model.ts::changeEntries
  - blizzard/web/projects/hub/src/app/admin/admin-shell.model.ts::changesTotal
---

# Move admin changes and history onto domains

Delivers [boundary.md](../spec/boundary.md#system)'s config-change/history read contract for `product:domain-boundary`,
under `decision:ui-toolkit/incremental-replacement`.

## Change

- Add the shared admin change/history domain and translators; move change list/detail, revision/diff models, and
  admin-shell count consumers off generated types.
- Preserve the existing queries, cache identities, pagination, grouping, actor labels, and revision presentation.
- Move their component/model specs onto domain fixtures; leave configuration writes unchanged.

## Acceptance

- Translator/model tests pin grouped and individual changes, empty and paginated history, totals, actor labels, field
  diffs, and revision ordering.
- Existing change navigation, record revision, and admin-shell count tests pass with domain fixtures.
- A source census finds no generated-type import in the migrated admin history/change/count consumers or their
  component/model specs; their boundary exemptions are removed.
- `web:typecheck`, `web:unit-test`, `web:structural-gate`, and affected admin browser layout checks pass.
