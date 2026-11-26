---
name: ws-connect-disconnect
depends-on:
  - target-patterns
  - epic:parallel-workspaces/environment-lifecycle
---

# The `ws connect` and `ws disconnect` commands

Adds `tracking::connect`, `tracking::disconnect`, and the `ws connect PATTERNS... FEATURE_BRANCH [--json]` and
`ws disconnect PATTERNS... [--json]` commands of [tracking.md](../spec/tracking.md): each selected non-pinned worktree's
own `branch.<head>.*` written or removed with no fetch and no ref check, then `push.default=upstream` through
`set_push_default_upstream` of [push-default.md](../../environment-lifecycle/spec/push-default.md), which
`epic:parallel-workspaces/environment-lifecycle` builds. Their rows are those of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md). Delivers
`product:worktree-upstream-control`'s connect bullets, the pinned and missing bullets of `product:select-git-targets`,
and `product:parallel-workspaces/upstream-push-default` for connect, under
`decision:parallel-workspaces/coordinated-git/remote-policy`,
`decision:parallel-workspaces/coordinated-git/pinned-skips`, and `decision:parallel-workspaces/partial-execution`.

## Change

- `crates/core/src/tracking.rs` holds `Changed`, `Stopped`, `TrackingRun`, `connect`, `disconnect`, and `TrackingError`:
  the load, the drift notice, selection with `EnvRule::Named`, then one worktree at a time; a pinned worktree is skipped
  and listed, and the first failed write stops the run with the rest unattempted.
- `crates/core/src/commands/ws/connect.rs` and `disconnect.rs` register the commands and write the shape A documents or
  the text of Result, exit 1 after the result when a write failed.
- `crates/core/topics/ws/connect.md` and `ws/disconnect.md`, referenced from the `ws` group topic.

## Acceptance

- Component tests through `InProcess` with real git: `ws connect 'alpha/*' feat/x` writes `branch.alpha.remote=origin`,
  `branch.alpha.merge=refs/heads/feat/x`, and `push.default=upstream` in each non-pinned worktree, for a branch origin
  does not have; the document lists them in `connected`, with `count`.
- A component test: a pinned worktree the pattern matches is not written and is listed in `skipped_pinned` and as
  "skipped (pinned)" in text; a pattern matching only pinned worktrees does not print "No worktrees matched".
- A component test: a detached worktree gets `branch.<env>.*` and the detached warning, and stays detached.
- A component test: `ws disconnect` unsets a connected worktree's upstream and lists an already disconnected one without
  writing.
- A component test: a config write failing on the second of three worktrees leaves the first written, names the second
  in `failed` and the third in `unattempted`, and exits 1.
- A component test: an absent worktree is warned, named in `missing`, and the run goes on.
- `cargo dev gate slice` exits 0.
