---
name: ws-merge
depends-on:
  - target-patterns
  - ref-tokens
  - fan-out
  - integration
  - ws-fetch
---

# The `ws merge` command

Adds `merge::merge`, `MergeEvent`, `MergeResult`, and the `ws merge SOURCE_REF [PATTERNS]...` command of
[merge.md](../spec/merge.md): the typed ref expanded per repository by `ref-tokens` before the first git call, then each
selected worktree and standalone integrated onto it through `integrate` of `integration`, on local refs only, as
`REMOTE_TASKS` groups of `fan-out`. A standalone held at a tag or commit is `resolve`d through
[standalone-pins.md](../../../../delivered/parallel-workspaces/reproducible-inputs/spec/standalone-pins.md) and skipped.
Delivers the arbitrary-ref bullets of `product:integrate-selected-repositories` and the ref-token bullet of
`product:select-git-targets`, under `decision:parallel-workspaces/coordinated-git/integration-conflicts` and
`decision:parallel-workspaces/coordinated-git/pinned-skips`, whose held skip is a registered deviation.

## Change

- `merge` takes `Scope` of `ws-fetch` ([refresh.md](../spec/refresh.md), Domain).
- `crates/core/src/merge.rs` holds `PinnedScope`, `MergeMode`, `MergeEvent`, `MergeResult`, `merge`, and `MergeError`; a
  missing source ref is `skipped-missing-ref`, a held standalone `held_pin`.
- `crates/core/src/commands/ws/merge.rs` registers the command with `--standalone`, `--all`, `--ff-only`, `--merge`,
  `--no-ff`, `--autostash`, `--exclude-pinned`, and `--only-pinned`, writes the events or their text rows, and exits 1
  unless every result is a success.
- `crates/core/topics/ws/merge.md`, referenced from the `ws` group topic.

## Acceptance

- Component tests through `InProcess`: `ws merge '{main}' '*/*'` merges each worktree's own main branch, two projects
  with different main branches included; no `git fetch` runs.
- A component test: `ws merge '{mian}' '*/*'` exits 1 with `RefToken` before `merge_started` and with no git call.
- Component tests: a ref missing in one repository is `skipped-missing-ref` there and merged elsewhere; `--no-ff`
  creates a merge commit; a conflicting merge is aborted and `diverged`.
- A component test: `--only-pinned` acts on pinned worktrees alone and `--exclude-pinned` on the rest, reporting each
  pinned worktree it leaves out as `skipped_pinned` with no git call in it and the run still a success.
- A component test: `--standalone` with an absent standalone warns it, names it in a `target_missing` event under
  `--json`, and starts no git process in it.
- A component test: with `--standalone`, a standalone held at a tag is `held_pin` and its HEAD does not move, and a
  branch-pinned one is merged with its lock record unchanged.
- `cargo dev gate slice` exits 0.
