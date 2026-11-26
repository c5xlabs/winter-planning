---
name: ws-status-command
depends-on:
  - status-snapshot
  - ws-fetch
---

# The `ws status` command

Adds the `ws status [PATTERNS]... [--json] [--fetch]` command of [status.md](../spec/status.md), Order, Document, Exit
code, and Text: `--fetch` running `ws fetch` of `ws-fetch` with `Scope::Project` first, then `collect` of
`status-snapshot`, then the v1 document or the per-environment tables written whole, then `exit_code`. Its row is that
of [command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), and its
exits extend
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
Exit codes. Delivers `product:inspect-workspace-git`'s output and `--fetch` bullets and the visible per-worktree
tracking of `product:worktree-upstream-control`, under `decision:command-vocabulary` and
`decision:parallel-workspaces/coordinated-git/fetch-freshness`.

## Change

- `crates/core/src/commands/ws/status.rs` registers the command; a collection error is `Termination::StatusCollection`,
  exit 2, with nothing more written.
- Text: per environment a `REPO | BRANCH | UPSTREAM | SYNC | DIRTY` table, showing `(detached)`, `(unborn)`, and
  `(not fetched)`, then the Projects, Standalones, and Workspace sections.
- `--fetch` under `--json` writes the fetch's shape C events, then the document; a failed refresh counts in `Unseen`.
- `crates/core/topics/ws/status.md`, referenced from the `ws` group topic.

## Acceptance

- The reference's `schemas/ws-status-v1.json` is copied into the test data, and a component test through `InProcess`
  validates every `--json` document against it, an unborn worktree's included.
- A component test: a detached worktree shows `(detached)`, an unborn one `<branch> (unborn)`, and an unfetched upstream
  `(not fetched)` in text.
- A component test: a clean, fully pushed workspace exits 0; a dirty matched worktree exits as results-and-exit-codes.md
  gives; an unknown literal environment exits 2 with nothing on stdout.
- A component test, per the `ws status` block of results-and-exit-codes.md: with PATTERNS matching a clean worktree, an
  orphan and a dirty source checkout leave the exit at 0; without PATTERNS, the same workspace exits 1.
- A component test: `--fetch` with the remote gone reports the failed refresh and exits 1; without `--fetch` no
  `git fetch` runs, as a recording program port shows.
- `cargo dev gate slice` exits 0.
