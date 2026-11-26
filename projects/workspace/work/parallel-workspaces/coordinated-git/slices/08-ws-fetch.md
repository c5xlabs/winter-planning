---
name: ws-fetch
depends-on:
  - target-patterns
  - fan-out
---

# Source sync and the `ws fetch` command

Adds `refresh::source_sync`, `refresh::fetch`, and the `ws fetch [PATTERNS]... [--standalone] [--all] [--json]` command
of [refresh.md](../spec/refresh.md), Source sync, `ws fetch`, and Events: each project with a selected worktree
refreshed once through its source checkout, its main fast-forwarded only when checked out there, and each standalone
fetched, as `REMOTE_TASKS` groups of `fan-out`. Every `git fetch` is `fetch` of
[standalone-pins.md](../../../../delivered/parallel-workspaces/reproducible-inputs/spec/standalone-pins.md), with its
Remote calls bounds. Delivers the refresh bullets of `product:integrate-selected-repositories`,
`engineering:remote-operation-bounds` for fetch, and the refresh half of `engineering:shared-git-effects`, under
`decision:parallel-workspaces/coordinated-git/fetch-freshness`.

## Change

- `crates/core/src/refresh.rs` holds `Scope`, `SourceSync`, `source_sync`, `FetchEvent`, `Label`, `fetch`, and
  `RefreshError`; a source checkout off its main is fetched and warned, not fast-forwarded; a main that cannot
  fast-forward fails the project's refresh.
- `crates/core/src/commands/ws/fetch.rs` registers the command, writes `target_missing`, the shape C events or their
  text rows, and exits 1 when a `repo_fetched` failed.
- `crates/core/topics/ws/fetch.md`, referenced from the `ws` group topic.

## Acceptance

- Component tests through `InProcess` with fixture remotes: two selected worktrees of one project cause one `git fetch`
  in its source checkout; the source main's advance is `repo_fetched` with its commit count; feature worktrees are
  untouched.
- Component tests: a source checkout on another branch is fetched and reports its warning with commits 0; a source main
  with a local-only commit that cannot fast-forward fails that project and the command exits 1.
- A component test: `--standalone` fetches each standalone and fast-forwards nothing.
- A component test: no project and no standalone writes "Nothing to fetch", nothing under `--json`, and exits 0.
- A component test: an unreachable remote fails its project with the fetch's `RepoError` while another project succeeds.
- `cargo dev gate slice` exits 0.
