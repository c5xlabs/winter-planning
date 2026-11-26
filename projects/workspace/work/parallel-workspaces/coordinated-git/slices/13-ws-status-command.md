---
name: ws-status-command
depends-on:
  - status-snapshot
  - ws-fetch
---

# The `ws status` command

Adds `status::run`, `status::document`, `status::text`, and the `ws status [PATTERNS]... [--json] [--fetch]` command of
[status.md](../spec/status.md), Order, Document, Exit code, and Text: `run` runs `--fetch`'s `ws fetch` of `ws-fetch`
with `Scope::Project` first, then `collect` of `status-snapshot` with the failed refreshes it counted, and hands back
the v1 document and the per-environment tables that `document` and `text` build, with `exit_code`; the command writes
the one its mode takes whole and exits with that code. Its row is that of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), and its exits
extend
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
Exit codes. Delivers `product:inspect-workspace-git`'s output and `--fetch` bullets and the visible per-worktree
tracking of `product:worktree-upstream-control`, under `decision:command-vocabulary` and
`decision:parallel-workspaces/coordinated-git/fetch-freshness`.

## Change

- `crates/core/src/status.rs` gains `run` and `StatusEnd`, as status.md, Domain and Order, give them: the fetch's events
  go to the sink, its failed `repo_fetched` results are counted and handed to `collect`, and its refusals and internal
  failure are the collection error.
- `crates/core/src/status.rs` gains `document` and `text`, `pub(crate)` like `run`, which calls them and carries their
  results and `exit_code` in `StatusEnd::Collected`: so `Snapshot.environments`, `ProjectRow.drift`, and the
  `environments` and `drift` keys stay out of the command file, as status.md, Domain, gives.
- `crates/core/src/commands/ws/status.rs` registers the command, makes its one call to `run`, and writes the document or
  the text `StatusEnd::Collected` carries and exits with its code, holding no key, field, or literal of the output;
  `StatusError::Config` is `Termination::Config`, exit 1, `StatusError::Collection` is `Termination::StatusCollection`,
  exit 2, with nothing more written, and `StatusEnd::Stopped` is the sink's ending.
- `text`: per environment a `REPO | BRANCH | UPSTREAM | SYNC | DIRTY` table, showing `(detached)`, `(unborn)`, and
  `(not fetched)`, then the Projects, Standalones, and Workspace sections.
- `--fetch` under `--json` writes the fetch's shape C events through the sink, then the document.
- `crates/core/topics/ws/status.md`, referenced from the `ws` group topic.

## Acceptance

- The reference's `schemas/ws-status-v1.json` is copied into the test data, and a component test through `InProcess`
  validates every `--json` document against it, an unborn worktree's included.
- A unit test: `text` of a snapshot with a detached worktree shows `(detached)`, an unborn one `<branch> (unborn)`, and
  an unfetched upstream `(not fetched)`; `document` of the same snapshot carries the `environments`, `drift`,
  `drift_missing`, `drift_undeclared`, `config_ref_drift`, and `head_drift` keys of status.md, Document.
- A component test: the text of `ws status` with a detached worktree shows `(detached)`.
- A component test: a worktree on a branch other than its environment's name is a row whose `branch` is that branch, the
  parked state of status.md, Environments.
- A component test: a clean, fully pushed workspace exits 0; a dirty matched worktree exits as results-and-exit-codes.md
  gives; an unknown literal environment exits 2 with nothing on stdout.
- A component test, per the `ws status` block of results-and-exit-codes.md: with PATTERNS matching a clean worktree, an
  orphan and a dirty source checkout leave the exit at 0; without PATTERNS, the same workspace exits 1.
- A component test: `--fetch` with the remote gone reports the failed refresh and exits 1; without `--fetch` no
  `git fetch` runs, as a recording program port shows.
- A component test: a configuration that does not parse exits 1 with its `ConfigError`, with and without `--fetch`, and
  nothing on stdout.
- `cargo dev gate slice` exits 0.
