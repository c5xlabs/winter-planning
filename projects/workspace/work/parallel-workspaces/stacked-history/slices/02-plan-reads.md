---
name: plan-reads
depends-on:
  - module-scaffold
  - epic:parallel-workspaces/coordinated-git
---

# The restack Git reads

Adds the reads of [restack-plan.md](../spec/restack-plan.md), Reads, to `restack.rs`: `tip`, `ancestor`, `fork`,
`rebasing`, and `branch`, beside the built `git::dirty`, each run through the program port of
[child-processes.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/child-processes.md) in the
worktree `<root>/<env>/<repo>` with its `git <subcommand>` span. The plan and the execution both read through them.
Serves `engineering:stack-boundary-integrity`'s rule that a tag never stands in for a local branch, and
`decision:parallel-workspaces/stacked-history/boundary-policy`'s fork point with no merge-base fallback. The branch
read's repo error is `TrackingRead` of [push.md](../../coordinated-git/spec/push.md), which
`epic:parallel-workspaces/coordinated-git` adds.

## Change

- `crates/core/src/restack.rs` gains `tip`, `ancestor`, `fork`, `rebasing`, and `branch`, `pub(crate)` so
  `restack/execute.rs` reads through them; `tip` reads a fully qualified ref, so `refs/heads/<env>` never resolves to a
  same-named tag.
- A missing worktree, or a directory that is no repository, reads as `None` or false in `tip`, `ancestor`, and `fork`;
  `branch` answers `None` on exit 1 and `GitFailure::Repo` with `TrackingRead` on any other exit.
- `restack.rs` names the `git merge-base` span its `ancestor` and `fork` calls open.
- No command calls them yet.

## Acceptance

- Component tests with real git: `tip` of `refs/heads/x` answers the branch's sha while a tag `x` points elsewhere, and
  `None` for a missing branch, a missing worktree, and a plain directory.
- Component tests: `ancestor` answers true for an ancestor, false for a non-ancestor, and false in a missing worktree.
- Component tests: after the predecessor branch is amended, `fork` answers the predecessor's old tip from its reflog,
  not the merge base; with the predecessor's reflog expired and the old tip unreachable from it, `fork` is `None`.
- Component tests: `rebasing` is true while a `git rebase` (merge backend) and a `git rebase --apply` are stopped on a
  conflict, and false otherwise.
- Component tests: `branch` answers the branch name on a branch, `None` when HEAD is detached, and `TrackingRead` in a
  directory that is no repository.
- A unit test over a recording tracer: each read opens its `git <subcommand>` span with `winter.repo`.
- `cargo dev gate slice` exits 0.
