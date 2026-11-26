---
name: ws-diff
depends-on:
  - target-patterns
---

# The `ws diff` command

Adds `diff::diff` and the `ws diff [PATTERNS]... [--staged | --branch] [--no-headers] [--json]` command of
[diff.md](../spec/diff.md): one worktree at a time, each `git diff` written with `a/<repo>/` and `b/<repo>/` prefixes so
the output names its repository and applies from the environment directory, with its numstat counts. Its row is that of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), and its
stdout the `ws diff` row of
[output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md). Delivers the
diff bullet of `product:inspect-workspace-git` and `engineering:git-observation-fidelity` for diffs.

## Change

- `crates/core/src/diff.rs` holds `DiffMode`, `RepoDiff`, `EnvDiff`, `diff`, and `DiffError`; the first failure ends the
  command with nothing on stdout.
- `crates/core/src/commands/ws/diff.rs` registers the command and writes the diffs, with environment and repository
  headers unless `--no-headers`, or the shape A document.
- `crates/core/topics/ws/diff.md`, referenced from the `ws` group topic.

## Acceptance

- Component tests through `InProcess` with real git: an unstaged change, a staged one, and a commit ahead of
  `origin/<main>` each appear in their mode alone, under their repository's prefix; an untracked file appears in none.
- A component test: the `--no-headers` output of two repositories applies with `git apply` from the environment
  directory of a clean copy.
- A component test: a file name holding a space is diffed and counted; a binary change counts 0 insertions and
  deletions.
- A component test: `--branch` with no `origin/<main>` exits 1 with nothing on stdout.
- A component test: `--json` holds `patterns`, `mode`, `results`, and `missing`, an absent worktree named there.
- `cargo dev gate slice` exits 0.
