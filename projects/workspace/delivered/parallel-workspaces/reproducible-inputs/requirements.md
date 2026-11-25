# Reproducible input requirements

For `epic:parallel-workspaces/reproducible-inputs`.

## Resolve standalone inputs intentionally (product:standalone-reference-policy)

An engineer can distinguish unpinned tracking, moving branch refs, and held tag or commit refs. Resolution order and
lock meaning are settled by `decision:parallel-workspaces/reproducible-inputs/resolution-and-locks`.

Acceptance:

- A reference resolves as an `origin` branch, then a tag, then a commit; an unresolvable reference is an error naming
  it, never a silent substitution of the default branch.
- A held reference does not move during the fetch of an update: a tag moved on its remote keeps its local tag.
- Reproducibility records are shareable separately from machine-local allocations and private configuration.

## Review intentional input updates (product:update-standalone-inputs)

An engineer can refresh selected standalone resolutions and review what changed without repinning unrelated inputs.

Acceptance:

- Selection takes literal standalone names and globs; a literal naming no pinned standalone is an error, and a glob
  matching none is a no-op.
- Each input reports up to date, re-pinned, or failed, and the change is reviewable as a lock diff.
- A tag moved on its remote is not followed: the local tag is kept, and re-pinning a lightweight tag reports it up to
  date.
- Re-pinning follows `decision:parallel-workspaces/reproducible-inputs/update-preservation`: a branch pin is reset to
  its `origin` branch, and uncommitted changes are refused unless temporary preservation is requested.
- Requested temporary preservation reports a failed restoration and where the preserved work remains.
- A failed input does not stop the others, and successful inputs keep their records.

## Freeze the checkout actually in use (product:freeze-standalone-inputs)

The engineer can record selected unpinned standalone checkouts as fixed inputs without requiring a network lookup.

Acceptance:

- Freeze records the current committed revision in the declaration's actual shared or local layer and its resolution.
- Already pinned inputs remain unchanged; missing or unreadable checkouts are refused by name.
- Dirty state requires explicit acknowledgement that the recorded commit does not describe uncommitted content.
- A local-only revision can be recorded.
- Freezing neither fetches nor changes the checkout and does not modify unrelated declarations or comments.

## Compare workspace definitions by content (product:workspace-content-identity)

An engineer can tell whether two workspace definitions have the same tracked content despite different commit history or
active feature work. The identity boundary is settled by
`decision:parallel-workspaces/reproducible-inputs/fingerprint-boundary`.

Acceptance:

- Comparison includes workspace tracked content and standalone identities and tracked content, including local edits.
- Staged additions, removals, renames, and subsequent working edits affect the result; untracked files do not.
- Project source content and feature-environment activity do not change definition identity by themselves.
- Equivalent content with different commit history or standalone declaration order compares equal.
- Missing or unreadable inputs are reported instead of silently omitted, and inspection leaves refs, index, and files
  unchanged without network access.
