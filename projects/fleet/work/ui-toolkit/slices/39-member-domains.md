---
name: member-domains
depends-on:
  - domain-boundary-gate
code:
  - blizzard/web/projects/hub/src/app/admin/members/members-page.ts::MembersPage
  - blizzard/web/projects/hub/src/app/admin/members/members-table.ts::MembersTable
  - blizzard/web/projects/hub/src/app/admin/members/members-page.model.ts::roleChoices
---

# Move member presentation onto domains

Delivers [boundary.md](../spec/boundary.md#system)'s member/role read contract for `product:domain-boundary`, under
`decision:ui-toolkit/incremental-replacement`.

## Change

- Translate member reads and role vocabulary at the boundary; move page/table inputs, action outputs, models, and
  fixtures onto domain contracts.
- Preserve hub-supplied assignable roles and revocable judgments, own-row restrictions, and existing role/revoke
  requests.
- Keep row-scoped pending state, confirmation, refusal messages, and settled invalidations unchanged; leave keyboard
  adoption to the grid bridge.

## Acceptance

- Translator/model tests cover own and other members, no assignable role, non-revocable members, and unforeseen role
  values without reconstructing hub authorization.
- Component tests preserve role-change and revoke requests, confirmation, refused changes, and per-row pending behavior.
- The members page, table, model, and component/model specs have no generated-type imports and leave boundary
  exemptions.
- `web:typecheck`, `web:unit-test`, `web:structural-gate`, and member-table browser layout checks pass.
