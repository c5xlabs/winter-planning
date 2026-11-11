# Stacked history requirements

For `epic:parallel-workspaces/stacked-history`.

## Declare a meaningful stack order (product:declare-stack-order)

The engineer can name dependent environments in a deliberate order and a base, independently of unordered target
patterns. Vocabulary follows `decision:command-vocabulary` and `decision:parallel-workspaces/target-selection`.

Acceptance:

- Help and preview show which environment is replayed onto which predecessor and which repositories participate.
- Chain members are actual local env branches, not same-named tags, globs, or an accidental repo-qualified operand.
- Repeated, missing, or inverted elements receive an actionable refusal rather than a successful all-skipped plan.
- Pinned projects are excluded, and a repository legitimately absent from one link is visibly nonparticipating.
- Base and explicit cut references can resolve each repository's own configured main without changing chain order.

## Establish replay boundaries before rewriting (product:review-replay-boundaries)

The engineer can inspect where each dependant's own work begins, including when a predecessor has been rewritten.
Boundary choices follow `decision:parallel-workspaces/stacked-history/boundary-policy`.

Acceptance:

- Preview names each link's boundary and provenance, not just the current predecessor tip.
- Amending a predecessor does not cause its old commits to be mistaken for the dependant's own work.
- Unknown boundaries refuse before any link moves; an explicit cut is validated rather than accepted as discard consent.
- A deliberate cut can address severed ancestry such as squash landing without dropping unrelated dependant work.
- Dirty tracked state, an ongoing rebase, or detached/parked HEAD is identified before rewriting begins.
- Dry-run is mutation-free and reports a plan, not guaranteed outcomes that depend on earlier links moving.

## Preserve dependant changes through a rewrite (product:replay-dependent-work)

The engineer can restack lower links before upper links so every dependant lands on its predecessor's newly established
history rather than its old tip.

Acceptance:

- Replay preserves the dependant's own changes and excludes predecessor changes outside the validated replay range.
- Upper links use the predecessor's post-rewrite position; a no-op lower link does not needlessly change its tip.
- A dependant with no work past its boundary can advance without inventing commits or losing attachment.
- Same-named tags do not change participation, boundaries, new bases, or which branch remains attached.
- A completed rewrite does not alter upstream tracking or implicitly push rewritten history.

## Resolve and resume a stopped stack rewrite (product:resume-stack-rewrite)

The engineer can resolve a conflict and continue without replaying completed links incorrectly. Resume behavior follows
`decision:parallel-workspaces/stacked-history/resume-policy` and `decision:parallel-workspaces/partial-execution`.

Acceptance:

- The first conflict stops later repositories and upper links and identifies the link, replayed commit, and paths.
- The conflicted repository remains in a recoverable paused state with manual continuation or abort guidance.
- Non-conflict failures retain completed outcomes and the unfinished tail instead of discarding the run report.
- Resuming recognizes completed work, including explicit-cut work already applied, without silently dropping ranges.
- Concurrent edits, lost boundary evidence, or a changed chain require fresh validation rather than blind continuation.
