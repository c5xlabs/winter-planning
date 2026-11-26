# Stacked history decisions

## Ref-token expansion before any Git call (decision:parallel-workspaces/stacked-history/token-expansion)

**Decider:** product owner and engineering

Restack expands BASE and `--cut` for every non-pinned repository before its first Git call, as
`product:parallel-workspaces/ref-tokens` requires, so a token error refuses the run with nothing read. This is a
registered deviation: the reference expands them during its reads and the cut only in participating repositories, so a
non-participating repository with no main branch configured now refuses the run where the reference ignored it.

## Boundary derivation and cut scope (decision:parallel-workspaces/stacked-history/boundary-policy)

**Decider:** product owner and engineering

Each link's boundary is Git's fork point from the predecessor's reflog, per link and repository, with no merge-base
fallback, and a link without one is refused; `--cut` sets the bottom link's boundary only, classified per repository as
applies, already done, or refused, and every boundary is frozen before anything moves. This keeps the reference tool's
behaviour, which already handles amended and squash-landed predecessors without guessing.

## Resume identity and lifetime (decision:parallel-workspaces/stacked-history/resume-policy)

**Decider:** product owner and engineering

Restack keeps no operation state: the engineer resolves a conflict with Git and re-runs the same command, completed
links then read as up to date, and a worktree already mid-rebase is refused. This keeps the reference tool's stateless
behaviour, so no continuation record can go stale or disagree with Git.
