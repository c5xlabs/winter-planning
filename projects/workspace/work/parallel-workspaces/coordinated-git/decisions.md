# Coordinated Git decisions

## Pinned and held work in push and merge (decision:parallel-workspaces/coordinated-git/pinned-skips)

**Decider:** product owner and engineering

Push reports skipped pinned commits whenever it skips them and reports an up-to-date candidate as up to date, connect
and disconnect list the pinned worktrees they skip, `ws merge --exclude-pinned` reports each pinned worktree it leaves
out, and push and `ws merge` skip a standalone held at a tag or commit with a reason. These are registered deviations:
the reference mentions skipped pinned commits only when nothing else was pushed, reports a published branch level with
its upstream but ahead of main as pushed, skips pinned worktrees in connect, disconnect, and `ws merge --exclude-pinned`
silently, counts a held standalone as a push failure, and lets `ws merge` move a held standalone off its locked commit.

## Ordinary integration conflicts (decision:parallel-workspaces/coordinated-git/integration-conflicts)

**Decider:** product owner and engineering

Integration keeps the reference tool's behavior: fast-forward-only first, `--merge` or `--rebase` as the fallback, a
conflicted merge or rebase always aborted and never left paused, and `--autostash` passed through as the only dirty-work
preservation. As a registered deviation, a failed abort is reported, and a dirty blocker, a failed autostash (a re-apply
that conflicts after Git exits 0 included), a Git lock another process holds, and a missing target ref are reported
apart from divergence or success, which the reference folds them into.

## Fetch effects and freshness (decision:parallel-workspaces/coordinated-git/fetch-freshness)

**Decider:** product owner and engineering

Refresh keeps the reference tool's mechanics: it goes through each project's source checkout, advances the source main
only when that checkout has it checked out, and pull still integrates on local refs after a failed refresh. As a
registered deviation, a failed refresh during pull or `ws status --fetch` is reported per project and fails the command,
where the reference only logs a warning.

## Remote and upstream policy (decision:parallel-workspaces/coordinated-git/remote-policy)

**Decider:** product owner and engineering

Workspace uses only `origin`, as the reference tool does. Connect writes `branch.<head>.remote=origin` and its merge ref
as given, or under the environment's branch name with a warning when the worktree is detached; push sends a non-pinned
worktree to its tracked branch on `origin`, creating it if needed, and pushes pinned projects and standalones plainly to
`origin`.
