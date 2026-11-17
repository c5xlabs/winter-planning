---
name: grid-bridge-members
external: github:paul-gross/blizzard#2283
depends-on:
  - domain-boundary-gate
code:
  - blizzard/web/projects/fleet/src/lib/kit/kit-select.ts::KitSelect
  - blizzard/web/projects/fleet/src/lib/kit/kit-button.ts::KitButton
  - blizzard/web/projects/hub/src/app/admin/members/members-table.ts::MembersTable
---

# Build the grid bridge and adopt it in Members

Delivers [choices.md](../spec/choices.md#interfaces)'s native-control bridge and its first interactive table for
`product:keyboard-choices` and `product:full-adoption`, retaining CDK select behavior.

## Change

- Add kit grid/row/cell/widget directives, native focus targets, managed tab indices, and event-routing markers.
- Compose the Members table with the bridge, preserving role-change and revoke actions and row-scoped pending state.
- Keep standalone buttons/selects unchanged and leave read-only tables out.

## Acceptance

- Real-browser tests use multiple rows, text cells, selects, and enabled/disabled buttons: one Tab entry, arrow
  movement, focus on the native control, and one activation per Enter/Space.
- Opening a select hands keys to its CDK popup; pick/Escape/dismissal deactivates and restores the same trigger.
- Row-scoped pending/permission restrictions still hold; no Aria import escapes the kit.
- `web:typecheck`, `web:unit-test`, `web:shell-sweep`, and `web:story-render` pass.
