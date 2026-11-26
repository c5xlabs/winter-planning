---
name: ws-push
depends-on:
  - target-patterns
  - worktree-probe
---

# The `ws push` command

Adds `push::push`, its report types, and the `ws push [PATTERNS]...` command of [push.md](../spec/push.md): one target
at a time, each in-scope worktree probed by `worktree-probe`, each candidate pushed to the branch its own configuration
tracks on `origin`, the source checkout's copy of that branch advanced only when it matched the pre-push remote tip, and
each standalone pushed or skipped. Every `git push` is a remote call of
[standalone-pins.md](../../../../delivered/parallel-workspaces/reproducible-inputs/spec/standalone-pins.md), Remote
calls. Delivers `product:push-worktree-destinations`, the push bullets of `product:worktree-upstream-control`,
`engineering:remote-operation-bounds` for push, and the remote-effect half of
`engineering:integration-recovery-evidence`, under `decision:parallel-workspaces/coordinated-git/remote-policy` and
`decision:parallel-workspaces/coordinated-git/pinned-skips`, whose rows are registered deviations.

## Change

- `crates/core/src/push.rs` holds `PinnedScope`, `PushStatus`, `LocalFastForward`, `PushOutcome`, `EnvPush`,
  `EnvSkipped`, `PushReport`, `push`, and `PushError`: the destination read from configuration only,
  `git push -u
  origin HEAD:refs/heads/<B>` for a non-pinned worktree and `git push origin` for a pinned one and a
  standalone.
- The local fast-forward uses `update-ref` with the old value when no worktree holds the branch, and `merge --ff-only`
  in a clean holder; any failure there is a skip reason and the row stays `pushed`.
- `crates/core/src/commands/ws/push.rs` registers the command with `--standalone`, `--all`, `--include-pinned`, and
  `--only-pinned`, writes the document or the text tables, and exits 1 unless the run succeeded.
- `crates/core/topics/ws/push.md`, referenced from the `ws` group topic.

## Acceptance

- Component tests through `InProcess` with a fixture remote: a connected worktree whose branch origin lacks creates it
  and is `pushed` with its count; a worktree with nothing ahead is `up_to_date` and runs no push; a worktree with
  commits and no upstream is `no_upstream` with the connect guidance and fails the run.
- Component tests: a pinned worktree with commits is a `skipped_pinned` row by default, beside pushed rows, and does not
  fail the run; with only pinned worktrees selected it is the environment skip and fails the run; `--include-pinned`
  pushes it.
- Component tests of the local copy: a source branch that matched the old remote tip advances by `update-ref`; one that
  had its own commit is skipped "not in sync"; a dirty holder is skipped "dirty"; each leaves the row `pushed`.
- A component test: a standalone held at a tag is `held_pin` and does not fail the run.
- A component test: a push to a remote that never answers, as a `GIT_SSH_COMMAND` that sleeps, ends `failed` within the
  Remote calls deadline under a short `WINTER_GIT_TIMEOUT_S`.
- `cargo dev gate slice` exits 0.
