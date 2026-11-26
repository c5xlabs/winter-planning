# Coordinated Git requirements

For `epic:parallel-workspaces/coordinated-git`.

## Select the Git work intended (product:select-git-targets)

The engineer can select environments, individual project worktrees, or standalone repositories without confusing their
scope. Patterns follow `product:target-patterns`, under `decision:parallel-workspaces/target-selection`.

Acceptance:

- Selection identifies the resolved target set, supports multi-target matching, and does not repeat overlapping matches.
- Project worktree selection is distinct from standalone and workspace-repository selection.
- Pinned-project inclusion is visible for every operation; feature tracking never silently rewires pinned projects.
- Refs name each repository's main through `product:parallel-workspaces/ref-tokens`.

## Track each worktree's own destination (product:worktree-upstream-control)

The engineer can connect or disconnect selected worktrees independently, including a feature branch not yet on a remote.

Acceptance:

- Connection records the intended target before first push without requiring its remote-tracking ref to exist.
- Inspection distinguishes disconnected, configured-but-unfetched, connected, detached, and parked states.
- Pull, push, and status use each worktree's own tracking, independent of sibling declaration order.
- Different upstreams within an environment are visible; a primary summary does not imply uniform tracking.
- A missing destination is refused or skipped with guidance, never inferred from another repository during push.
- Connect and push use only `origin`, under `decision:parallel-workspaces/coordinated-git/remote-policy`; connecting a
  detached worktree records the environment's branch and warns that the worktree stays detached.

## Refresh and integrate selected repositories (product:integrate-selected-repositories)

The engineer can refresh remote refs, integrate a tracked upstream, or merge a chosen local ref without an implicit
destructive reset. Refresh follows `decision:parallel-workspaces/coordinated-git/fetch-freshness` and integration
`decision:parallel-workspaces/coordinated-git/integration-conflicts`.

Acceptance:

- Fetch refreshes each project with a selected worktree on disk once, through its source checkout, and reports any
  advance of the source checkout's main separately from feature-worktree integration.
- A source main that cannot fast-forward fails that project's fetch.
- Pinned projects integrate their own main.
- Integration is fast-forward-only unless `--merge` or `--rebase` is given; `--autostash` is the way to carry dirty work
  through it.
- Arbitrary-ref integration requires an explicit target selection and can operate without an implicit network refresh.
- A dirty blocker, a failed autostash, and a missing target ref are each reported apart from divergence.
- A failed refresh is reported for its project and fails the command, even when pull goes on to integrate on local refs.
- A conflicted merge or rebase is always aborted, never left paused, and a failed abort is reported.
- A standalone's held tag or commit reference does not move during fetch, pull, or `ws merge`, and `ws merge` skips it
  with a reason; a branch pin follows the pull's integration, and its record advances only when the pull fast-forwards
  it, under `decision:parallel-workspaces/reproducible-inputs/resolution-and-locks`.

## Push only the intended work (product:push-worktree-destinations)

The engineer can publish committed work to each selected worktree's own destination without silently including pinned
projects or rewriting a local integration branch that has independent work.

Acceptance:

- Publication uses the selected worktree's intended revision and destination, including creation of a remote branch.
- Pinned projects are excluded unless deliberately included, and skipped pinned commits are reported whenever they are
  skipped, under `decision:parallel-workspaces/coordinated-git/pinned-skips`.
- Up-to-date targets are distinguishable from targets not attempted or lacking an upstream.
- A standalone held at a tag or commit is skipped with a reason, not reported as a failed push.
- Optional local landing-branch advancement requires it to have matched the pre-push remote tip and preserves dirty,
  ahead, behind, or diverged copies rather than resetting them.
- A failed follow-up local advancement does not claim the successful remote push failed or hide the local skip.

## Inspect Git state without changing it (product:inspect-workspace-git)

The engineer or agent can inspect status and changes across repositories without fetching, rewriting tracking, or
creating setup state by default.

Acceptance:

- Status separates main-relative and upstream-relative divergence, staged, unstaged, and untracked changes.
- A missing or non-repository worktree, a failed probe, detached HEAD, and unborn history are not shown as healthy clean
  state.
- Diffs distinguish unstaged, staged, and branch-relative changes with repository and environment attribution.
- `ws status --fetch` reports a failed refresh per project and fails, unlike read-only inspection, which never fetches.
- Structured output and success categories follow `decision:command-vocabulary`; scoped cleanliness is distinguishable
  from unrelated global drift shown for context.
- Each pinned standalone's declared reference, recorded revision, resolution kind, and checkout drift are inspectable,
  under `decision:parallel-workspaces/reproducible-inputs/resolution-and-locks`.
