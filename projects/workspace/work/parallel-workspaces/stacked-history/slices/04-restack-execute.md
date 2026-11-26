---
name: restack-execute
depends-on:
  - module-scaffold
  - restack-plan
---

# Replaying a restack plan

Adds `Outcome`, `Completed`, `Conflict`, `Failure`, `Stop`, `Report`, and `execute` of
[restack-execution.md](../spec/restack-execution.md), Domain, Replay, and Resume: each link bottom first, each
participating repository in configuration order, its new base read afresh, an up-to-date repository left alone, the rest
replayed with `git rebase --onto`, and the run stopped at the first conflict or failure with what completed and what
remains. A rebase that fails carries `GitOperation::Rebase` of `module-scaffold`, in the `git rebase` span. Delivers
`product:replay-dependent-work`, `product:resume-stack-rewrite`, `engineering:stack-stop-classification`, and
`engineering:stack-preservation-evidence`, under `decision:parallel-workspaces/stacked-history/resume-policy` and
`decision:parallel-workspaces/partial-execution`.

## Change

- `crates/core/src/restack/execute.rs` holds the report types and `execute`; the cut exemption, the up-to-date test, and
  the rebase are those of Replay, with `<env>` passed bare so the branch stays attached.
- A non-zero rebase is a `Conflict` only when git left `stopped-sha` or `original-commit`, the repository left
  mid-rebase; otherwise, or when a classification read fails, a `Failure` with the repo error.
- `restack/execute.rs` names the `git rebase` span its rebase opens.
- Nothing is written for a later run, nothing is pushed, and no upstream setting changes.
- `execute` takes the workspace root and the links of `Plan::Links`, and reads each repository's new base from its link
  repository's `pred_ref`, so the bottom link's BASE is not expanded again.
- A git that cannot be started, in a read or the rebase, is a `Failure` carrying that error's message.
- `execute` and the report types are re-exported by name from `lib.rs` for its tests, `Conflict` as `RestackConflict`,
  since `agents_md::Conflict` holds `Conflict` at the crate root ([restack-execution.md](../spec/restack-execution.md),
  Domain); no command calls them yet.

## Acceptance

- Component tests with real git over two repositories: with the bottom predecessor amended, a two-link chain rebases the
  bottom env onto the new predecessor and the upper env onto the bottom env's rewritten tip; each env's commits past its
  new base have exactly its own subjects and content, none of the predecessor's old commits.
- Component tests: a link already on its predecessor is `UpToDate` and its tip does not change; an env with no commits
  past its boundary is `Rebased` onto the new base and stays on `refs/heads/<env>`.
- A component test: a squash-landed lower branch with `--cut` at its old tip replays only the dependant's own commits
  onto BASE.
- A component test: a same-named tag pointing elsewhere is unmoved and the env branch stays attached after `Rebased`.
- A component test: a conflict in the second repository of the bottom link reports its link, the replayed commit, and
  the conflicted paths; the first repository's `Rebased` is kept; that repository stays mid-rebase; the third repository
  and the upper link keep their tips; `remaining` starts at the bottom link.
- A component test: a rebase failing with `index.lock` held is a `Failure` carrying `Rebase` and git's stderr, not a
  `Conflict`, with the completed outcomes kept.
- A component test: a predecessor branch deleted between planning and its link is a `Failure` naming that ref.
- A unit test over a recording program port: a git that cannot be started during a link's reads is a `Failure` carrying
  that error's message, with the completed outcomes kept.
- A component test: BASE `{main}` over two repositories whose `main_branch` values differ rebases each bottom env onto
  its own repository's main branch.
- Component tests of resume, planning and executing again: after `git rebase --continue` the completed links and the
  resolved repository read `UpToDate` and are not rebased again; a cut repository already replayed reads `UpToDate`; a
  cut already contained in BASE re-reports `Rebased` with its tip unchanged.
- A component test: `branch.<env>.remote` and `branch.<env>.merge` are unchanged after a completed run, and a recording
  program port shows no `git push`.
- `cargo dev gate slice` exits 0.
