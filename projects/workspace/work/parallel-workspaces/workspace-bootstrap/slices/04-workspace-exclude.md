---
name: workspace-exclude
depends-on:
  - managed-blocks
  - module-scaffold
---

# The workspace exclude file: locating it, writing a block, and finalizing

Adds `locate`, `locate_for_write`, `write_block`, and `finalize` of [managed-content.md](../spec/managed-content.md),
Interfaces, The workspace exclude file, with the git calls they make. Delivers `engineering:workspace-exclude-isolation`
and the exclude half of `product:managed-content-ownership`: independent excludes for a linked-worktree workspace, no
configuration write on a read, and removal of only a target's own block.

## Change

- `crates/core/src/workspace_exclude.rs` holds the four entries over `Ports`; `git` calls
  `rev-parse
  --absolute-git-dir`, `rev-parse --git-common-dir`, and the `git config` reads and writes the spec lists,
  each through the program port with its operation span.
- `finalize` writes the `winter-config` block whatever the mode, and with any mode but `none` strips each slash-free
  block not eligible and not kept, then writes each eligible block, in one update of the file.
- They are re-exported from `lib.rs`; no command calls them yet.

## Acceptance

- Component tests with real git in the isolated tree: a plain clone's file is `<root>/.git/info/exclude`; a root without
  `.git` is `NotARepository` and nothing is written; a root that is a linked worktree of a normal repository and one of
  a bare common directory each get `extensions.worktreeConfig` and their own `core.excludesFile`, `core.bare` and
  `core.worktree` moved to the common `config.worktree`, and a sibling worktree's `git status` still works.
- A component test: a second `locate_for_write` on a configured worktree runs no `git config` write (the stub records
  the calls); `locate` never writes.
- Component tests of `finalize`: with two eligible standalones then one, the removed one's block is stripped and the
  other kept; with none, the last block is stripped and `winter-config` stays; `winter-dir/projects` survives every
  call; a name in `keep` survives; mode `none` writes `winter-config` alone, leaves other blocks, and returns
  `Unchanged`; a run that adds `winter-config` alone returns `Unchanged`; a user line outside every block is unchanged.
- A component test: `write_block` reports `None` when the block already holds its lines.
- `cargo dev gate slice` exits 0.
