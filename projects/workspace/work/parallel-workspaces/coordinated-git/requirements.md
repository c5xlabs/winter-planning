# Coordinated Git requirements

For `epic:parallel-workspaces/coordinated-git`.

## Select the Git work intended (product:select-git-targets)

The engineer can select environments, individual project worktrees, or standalone repositories without confusing their
scope. Grammar and defaults follow `decision:parallel-workspaces/target-selection`.

Acceptance:

- Selection identifies the resolved target set, supports multi-target matching, and does not repeat overlapping matches.
- Project worktree selection is distinct from standalone and workspace-repository selection.
- Pinned-project inclusion is visible for every operation; feature tracking never silently rewires pinned projects.
- Unknown targets, absent locations, and an empty match have distinguishable outcomes.
- Ref tokens for each repository's main resolve independently; invalid tokens refuse before requested mutations.

## Track each worktree's own destination (product:worktree-upstream-control)

The engineer can connect or disconnect selected worktrees independently, including a feature branch not yet on a remote.

Acceptance:

- Connection records the intended target before first push without requiring its remote-tracking ref to exist.
- Inspection distinguishes disconnected, configured-but-unfetched, connected, detached, and parked states.
- Pull, push, and status use each worktree's own tracking, independent of sibling declaration order.
- Different upstreams within an environment are visible; a primary summary does not imply uniform tracking.
- A missing destination is refused or skipped with guidance, never inferred from another repository during push.
- Supported non-origin and local upstream behavior is explicit under
  `decision:parallel-workspaces/coordinated-git/remote-policy`.

## Refresh and integrate selected repositories (product:integrate-selected-repositories)

The engineer can refresh remote refs, integrate a tracked upstream, or merge a chosen local ref without an implicit
destructive reset. Freshness and execution policy follow the decisions for this epic and initiative.

Acceptance:

- Fetch identifies any source-checkout advancement as an effect, separately from feature-worktree integration.
- Pull names the effective upstream per target; pinned projects integrate their own main.
- Fast-forward-only is available; deliberate merge or rebase modes and dirty-state preservation are explicit choices.
- Arbitrary-ref integration requires an explicit target selection and can operate without an implicit network refresh.
- Missing refs, divergence, dirty blockers, authentication failure, and unavailable remotes are not conflated.
- Failure to refresh cannot be disguised as verified current remote state; any stale-ref continuation is disclosed.
- Conflict outcomes identify whether Git was restored or remains paused, and where preserved dirty work remains.

## Push only the intended work (product:push-worktree-destinations)

The engineer can publish committed work to each selected worktree's own destination without silently including pinned
projects or rewriting a local integration branch that has independent work.

Acceptance:

- Publication uses the selected worktree's intended revision and destination, including creation of a remote branch.
- Pinned projects are excluded unless deliberately included, and skipped committed work is visible.
- Up-to-date targets are distinguishable from targets not attempted or lacking an upstream.
- Optional local landing-branch advancement requires it to have matched the pre-push remote tip and preserves dirty,
  ahead, behind, or diverged copies rather than resetting them.
- A failed follow-up local advancement does not claim the successful remote push failed or hide the local skip.

## Inspect Git state without changing it (product:inspect-workspace-git)

The engineer or agent can inspect status and changes across repositories without fetching, rewriting tracking, or
creating setup state by default.

Acceptance:

- Status separates main-relative and upstream-relative divergence, staged, unstaged, and untracked changes.
- Missing repositories, failed probes, detached HEAD, and unborn history are not fabricated as healthy clean state.
- Diffs distinguish unstaged, staged, and branch-relative changes with repository and environment attribution.
- Explicit refresh is distinguishable from read-only inspection and reports its additional effects.
- Structured output and success categories follow `decision:command-vocabulary`; scoped cleanliness is distinguishable
  from unrelated global drift shown for context.
