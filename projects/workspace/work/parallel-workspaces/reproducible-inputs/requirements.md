# Reproducible input requirements

For `epic:parallel-workspaces/reproducible-inputs`.

## Resolve standalone inputs intentionally (product:standalone-reference-policy)

An engineer can distinguish unpinned tracking, moving branch refs, and held tag or commit refs. Ambiguity and lock
meaning are settled by `decision:parallel-workspaces/reproducible-inputs/resolution-and-locks`.

Acceptance:

- Declared reference, resolved revision, resolution kind, and checkout drift are inspectable for each standalone.
- Ambiguous or unresolvable references are explained rather than silently substituted with the default branch.
- Held references do not move during ordinary fetch or pull; moving references follow an explicit integration policy.
- Reproducibility records are shareable separately from machine-local allocations and private configuration.

## Materialize recorded resolutions safely (product:recorded-input-materialization)

The engineer can materialize a recorded workspace input without silently re-resolving it or discarding checkout work.

Acceptance:

- A valid matching recorded resolution has the documented precedence over a newly moved remote reference.
- Missing, stale, corrupt, unsupported, or unavailable resolution records have distinct outcomes and recovery guidance.
- Materialization protects dirty state and local-only commits; skipping a prompt is not discard consent.
- Failures do not report an unapplied resolution as the checkout's actual state or remove other inputs' records.

## Review intentional input updates (product:update-standalone-inputs)

An engineer can refresh selected standalone resolutions and review what changed without repinning unrelated inputs.

Acceptance:

- Selection supports explicit standalone identities and stated multi-target matching, with unknown literals diagnosed.
- Updates distinguish unchanged, advanced, held, refused, and failed inputs and surface a reviewable resolution change.
- Branch integration does not abandon local commits through an implicit reset; deliberate repinning obeys an explicit
  preservation policy under `decision:parallel-workspaces/reproducible-inputs/update-preservation`.
- Requested temporary preservation of dirty state reports restoration conflicts and where the preserved work remains.
- Partial failure preserves successful input records and reports unattempted or unresolved inputs.

## Freeze the checkout actually in use (product:freeze-standalone-inputs)

The engineer can record selected unpinned standalone checkouts as fixed inputs without requiring a network lookup.

Acceptance:

- Freeze records the current committed revision in the declaration's actual shared or local layer and its resolution.
- Already pinned inputs remain unchanged; missing or unreadable checkouts are refused by name.
- Dirty state requires explicit acknowledgement that the recorded commit does not describe uncommitted content.
- A local-only revision can be recorded, but its lack of demonstrated availability to another machine is not hidden.
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
