# Coordinated Git decisions

## Remote and upstream policy (decision:parallel-workspaces/coordinated-git/remote-policy)

**Decider:** product owner and engineering

**Open.** Choose non-origin remotes, local upstreams, and destination changes on detached or parked worktrees; this
blocks finalizing `product:worktree-upstream-control` and `product:push-worktree-destinations` without borrowing an
environment-wide branch as a fallback.

## Fetch effects and freshness (decision:parallel-workspaces/coordinated-git/fetch-freshness)

**Decider:** product owner and engineering

**Open.** Choose whether refresh advances source checkouts, how parked source branches are treated, and whether pull may
continue against stale refs after failed refresh; this blocks finalizing `product:integrate-selected-repositories` and
explicit-refresh behavior in `product:inspect-workspace-git`.

## Ordinary integration conflicts (decision:parallel-workspaces/coordinated-git/integration-conflicts)

**Decider:** product owner and engineering

**Open.** Choose abort-versus-pause behavior for ordinary merge/rebase conflicts and supported dirty-work preservation
policy, separately from deliberate stack rewrites; this blocks finalizing `product:integrate-selected-repositories` and
`engineering:integration-recovery-evidence`.
