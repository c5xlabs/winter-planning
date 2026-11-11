# Safe history requirements

For `epic:parallel-workspaces/safe-history`.

## Review destructive scope before execution (product:destructive-target-review)

The engineer can distinguish branch adoption, pointer reset, and untracked deletion before approving effects. Selection
and approval follow `decision:parallel-workspaces/target-selection` and
`decision:parallel-workspaces/destructive-approval`.

Acceptance:

- The command explains whether it affects a whole environment or selected worktrees and names pinned exclusions.
- Preview identifies concrete targets, intended refs or deletion paths, and refusal reasons without mutation.
- Missing refs, invalid tokens, ambiguous raw revisions across independent repositories, and unsafe paths refuse before
  execution; discard consent does not make an invalid destination valid.
- Unattended structured use requires explicit scope and approval rather than treating output format as consent.
- A changed target or loss set after preview is handled according to a stated policy, not undisclosed expansion.

## Adopt a feature branch without hiding abandonment (product:adopt-feature-history)

The engineer can place an environment on the intended feature branch and connect its repositories without silently
discarding unpublished work. Adoption scope follows `decision:parallel-workspaces/safe-history/adoption-policy`.

Acceptance:

- Each repository resolves its own feature or main; a branch unknown everywhere requires explicit new-branch intent.
- Missing both target and fallback refuses even when discard is approved; no implicit fetch changes the evidence.
- Default preflight protects tracked dirt and commits absent from the worktree's own upstream or stated fallback.
- Protection also covers the env-named branch that will move when HEAD is detached or parked on a different branch.
- Successful adoption attaches the intended env branch before establishing its new tracking; pinned projects are not
  connected to the feature target.
- A failure before a target moves does not leave it deceptively connected to the requested new branch.

## Reset pointers without changing tracking (product:reset-worktree-pointers)

The engineer can move selected worktree pointers using soft, mixed, or hard reset semantics without also adopting a new
branch destination.

Acceptance:

- Soft preserves index and working files; mixed resets index but keeps working files; hard resets all three Git trees.
- Every mode preserves tracking and attachment state, including detached HEAD.
- Hard reset protects tracked dirt and unpublished commits by default, measuring abandonment against the old upstream,
  not the new target; soft/mixed do not claim to preserve the old branch position.
- Per-repository main tokens resolve independently, and missing refs refuse the validated target set before effects.
- A bare revision identifier with no shared cross-repository meaning cannot silently reset several independent repos.
- Force, if offered, identifies which preservation checks it bypasses without expanding scope.

## Remove only intended untracked content (product:remove-untracked-content)

The engineer can preview and remove untracked content without deleting ignored provisioned artifacts or confusing this
operation with project-declared resource cleanup.

Acceptance:

- Preview and removal use matching Git selection semantics, including empty directories and nested repositories.
- Ignored files and tracked modifications remain; there is no bulk ignored-file removal hidden behind ordinary force.
- Approval shows deletion paths and warns that deleted untracked content has no reflog recovery.
- Results name actual removals, including paths removed before a partial failure, rather than merely echoing a preview.
- A failed target stops or continues only under the initiative's stated execution policy, retaining earlier outcomes.
- Top-level lifecycle cleanup and Git untracked cleanup remain distinguishable in help and results.

## Recover after a destructive operation stops (product:history-operation-recovery)

The engineer can distinguish a refused preflight from an operation that already changed refs or deleted files. Recovery
guarantees follow `decision:parallel-workspaces/partial-execution`.

Acceptance:

- A preflight refusal causes no requested ref, tracking, index, or file mutation within its validated scope.
- A mid-execution failure names completed, failed, and unattempted targets and does not claim cross-repo rollback.
- Pointer-changing outcomes identify recovery information for moved refs; deletion outcomes never imply recoverability.
- Concurrent Git changes or interruption do not produce an unqualified success based solely on earlier checks.
- Guidance describes the actual retained HEAD, tracking, and working state rather than a presumed clean final state.
