---
name: worktree-probe
depends-on:
  - module-scaffold
---

# The worktree status probe

Adds the worktree probe of [status.md](../spec/status.md), Environments, steps 1 to 3, with `WorktreeRow`: the
`--porcelain=v2 -z` status read into branch, upstream, tracking counts and presence, staged, unstaged, untracked, and
dirty counts, the main-relative ahead and behind, and the last commit subject, with unborn history told apart. It
extends the built probe of
[location-listing.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/location-listing.md), The
worktree probe. `ws status` and `ws push` both read it. Delivers `engineering:git-observation-fidelity` and the
per-worktree state of `product:inspect-workspace-git` and `product:worktree-upstream-control`.

## Change

- `crates/core/src/status.rs` holds `WorktreeRow` without its `nested` field, and `Probed` and `probe`, which fill one
  from a worktree path, through the program port with `TrackingAheadBehindProbe` and `MainAheadBehindProbe` spans.
- The probe parses records rather than human status text: type 1, 2, u, and `?` records counted as the spec gives, a
  rename counted once by its new path.
- An unborn worktree runs neither the main-relative probe nor the log, and is marked so the text can show it.
- `WorktreeRow`, `Probed`, and `probe` are re-exported by name from `lib.rs` for its tests
  ([status.md](../spec/status.md), Domain); no command calls them yet.

## Acceptance

- Component tests with real git: a file name holding a space, a newline, and a tab is counted once; a rename counts one
  staged path; a partly staged file counts once in staged and once in unstaged and once in dirty; an unmerged path
  counts in staged and unstaged.
- Component tests: a detached worktree has `branch` null; a branch tracking an unfetched upstream has `upstream` set,
  `tracking_ref_present` false, and tracking counts 0; a fetched one has its counts and true.
- Component tests: main-relative ahead and behind come from `origin/<main>...HEAD`, are 0, 0 when `origin/<main>` does
  not resolve, and are the probe's `RepoError` for another failure; `last_commit_subject` is null at ahead 0.
- A component test: an unborn worktree answers ahead and behind 0, `last_commit_subject` null, and its unborn mark,
  without an error.
- `cargo dev gate slice` exits 0.

## Out of scope

- The `nested` member belongs to `epic:parallel-workspaces/nested-workspaces`
  ([child-state.md](../../nested-workspaces/spec/child-state.md)).
