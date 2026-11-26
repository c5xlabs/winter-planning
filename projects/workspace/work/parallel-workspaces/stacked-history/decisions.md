# Stacked history decisions

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
