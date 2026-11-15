---
name: project-forms
depends-on:
  - config-dialog-seam
code:
  - blizzard/web/projects/hub/src/app/admin/projects/projects-page.ts::ProjectsPage
  - blizzard/web/projects/hub/src/app/admin/projects/project-link-dialog.ts::ProjectLinkDialog
  - blizzard/web/projects/hub/src/app/admin/projects/project-branch-dialog.ts::ProjectBranchDialog
---

# Migrate project configuration forms

Moves project create/edit, record linking, and link-branch editing to [forms.md](../spec/forms.md#system)'s
project-owned domains and services. Serves `product:full-adoption`, `product:error-at-field`, and
`product:domain-boundary`.

## Change

- Use typed project/link drafts and field-tree views while preserving permissions, link choices, and branch inheritance.
- Use the project revision for project edits and the link's own revision for branch edits.
- Translate project reads and move the feature's existing views off generated types.

## Acceptance

- Tests cover create, sparse edit, linking, explicit branch override/inheritance, and empty link choices.
- A query refresh cannot replace an open edit's baseline; stale refusals retain its draft.
- Invalid fields send nothing and hub field refusals focus their bound controls; `web:typecheck` and `web:unit-test`
  pass.
- The boundary gate finds no unexempted project-feature import, and all migrated project units leave the exemption list.
