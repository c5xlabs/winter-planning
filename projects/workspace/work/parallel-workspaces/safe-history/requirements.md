# Safe history requirements

For `epic:parallel-workspaces/safe-history`.

## Review destructive scope before execution (product:destructive-target-review)

The engineer can distinguish branch adoption, pointer reset, and untracked deletion before approving effects. Selection
follows `product:target-patterns` and approval follows `decision:parallel-workspaces/destructive-approval`.

Acceptance:

- Help says whether a command acts on a whole environment or on the worktrees its patterns select; pinned projects are
  never touched.
- Reset and clean dry-runs name each selected worktree with its resolved ref or deletion paths, and any refusal, without
  mutation.
- Missing refs, unknown ref tokens (`product:parallel-workspaces/ref-tokens`), and a bare revision across several
  worktrees refuse before execution; `--force` does not make an invalid destination valid.
- Unattended structured use requires explicit scope and approval rather than treating output format as consent.
- One `--force` approves the prompt and the discards the command names; it never widens the selection.

## Adopt a feature branch without hiding abandonment (product:adopt-feature-history)

The engineer can place an environment on the intended feature branch and connect its repositories without silently
discarding unpublished work. Adoption follows `decision:parallel-workspaces/safe-history/adoption-policy`.

Acceptance:

- Each non-pinned repository moves to the feature branch, or to its own main where the feature branch is absent while
  still tracking the feature branch; a branch no repository knows requires `--new`.
- Missing both target and fallback refuses even under `--force`; no implicit fetch changes the evidence.
- Without `--force`, a repository with staged or unstaged tracked changes or unpushed commits refuses the whole command;
  untracked files alone never refuse, and a check that cannot run refuses rather than reading as nothing at risk.
- Protection also covers the env-named branch that will move when HEAD is detached or parked on a different branch.
- Without `--force`, an untracked file in the way of the target refuses the whole command before any repository moves,
  and stays on disk; with `--force` it is overwritten.
- Successful adoption attaches the intended env branch before establishing its new tracking; pinned projects are not
  connected to the feature target.
- Adopting `{main}` connects each env branch to its own main, so a later push goes to main.
- A failure before a target moves does not leave it deceptively connected to the requested new branch.

## Reset pointers without changing tracking (product:reset-worktree-pointers)

The engineer can move selected worktree pointers using soft, mixed, or hard reset semantics without also adopting a new
branch destination.

Acceptance:

- Soft preserves index and working files; mixed resets index but keeps working files; hard resets all three Git trees.
- Every mode preserves tracking and attachment state, including detached HEAD.
- Untracked files survive every mode, except that hard reset overwrites one in the way of a tracked file at the target.
- Hard reset refuses tracked dirt and unpushed commits by default, measured as adoption measures them against the old
  upstream, not the new target; soft and mixed do not claim to preserve the old branch position.
- The ref resolves in each repository on its own, and a ref missing in any selected repository refuses the whole run
  before effects.
- A bare hex revision of 4 to 40 characters is refused when more than one worktree is selected, even under `--force`;
  `refs/heads/<name>` names an all-hex branch.
- `--force` skips the prompt and the dirt and unpushed checks, never the ref refusals or the selection.

## Remove only intended untracked content (product:remove-untracked-content)

The engineer can preview and remove untracked content without deleting ignored provisioned artifacts or confusing this
operation with project-declared resource cleanup.

Acceptance:

- Preview and removal use matching Git selection semantics, including empty directories and nested repositories.
- Ignored files and tracked modifications remain; there is no bulk ignored-file removal hidden behind ordinary force.
- Approval shows deletion paths and warns that deleted untracked content has no reflog recovery.
- Results name actual removals, including paths removed before a partial failure, rather than merely echoing a preview.
- A failed worktree stops the run with its cause and every path removed so far; later worktrees stay untouched.
- Top-level lifecycle cleanup and Git untracked cleanup remain distinguishable in help and results.

## Recover after a destructive operation stops (product:history-operation-recovery)

The engineer can distinguish a refused preflight from an operation that already changed refs or deleted files. Recovery
guarantees follow `decision:parallel-workspaces/partial-execution`.

Acceptance:

- A preflight refusal causes no requested ref, tracking, index, or file mutation within its validated scope.
- A mid-execution failure names completed, failed, and unattempted targets and does not claim cross-repo rollback.
- Each moved pointer's result names the commit it pointed at before; deletion outcomes never imply recoverability.
- Guidance describes the actual retained HEAD, tracking, and working state rather than a presumed clean final state.
