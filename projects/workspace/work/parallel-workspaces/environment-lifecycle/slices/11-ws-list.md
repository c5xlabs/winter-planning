---
name: ws-list
depends-on:
  - allocation-rule
  - worktree-git-operations
---

# The `ws list` command

Adds `env_list::list` and the `ws list [--json]` command of [list.md](../spec/list.md): the environments present, each
with its index, path, feature branch, and distinct remote count, after the drift notice of
[drift-warning.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/drift-warning.md) and the registry
findings of [allocation.md](../spec/allocation.md), Findings. Its row is that of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), Commands, and
its JSON the shape A array of
[output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md). Delivers the
listing and inspection bullets of `product:environment-targets`, the reporting bullets of
`product:stable-environment-allocation` and `product:environment-state-recovery`, and the drift wiring engineering.md
names, under `decision:parallel-workspaces/environment-lifecycle/discovery-authority`.

## Change

- `crates/core/src/env_list.rs` holds `Listed` and `list`: `environments::present` as built, each environment's index
  from the built `env_index::lookup`, and its branches from `head_branch` and `setting` over the non-pinned worktrees.
- `crates/core/src/commands/ws/list.rs` registers `ws list [--json]`: the load, the drift notice, the registry read, the
  findings as warnings, then the JSON document or the `ENV | FEATURE BRANCH | STATUS` table through `output::table`.
- `crates/core/topics/ws/list.md`, referenced from the `ws` group topic.

## Acceptance

- A component test through `InProcess`: two environments are listed in ascending index with `<root>/<name>` as path; the
  JSON document holds the fields list.md gives, `extensions` `{}`, and the text table one row each.
- Component tests of the feature branch: the first non-pinned worktree whose branch has remote `origin` and merge
  `refs/heads/<b>` gives `<b>`, a connected never-pushed branch included; a detached worktree, a non-origin remote, and
  an absent worktree give none, and the absent one writes no warning; two different upstreams give a distinct remote
  count of 2.
- A component test: a registry holding an out-of-band index, a name on an alias slot, an alias off its position, and an
  index two names share writes one warning each, by name, and exits 0.
- A component test: a state file that does not parse exits 1 with nothing on stdout.
- A component test: with no environment present, the text is the header row alone and `--json` writes `[]`.
- A component test: a run leaves every file under the workspace unchanged and starts no git process that writes, as a
  recording program port shows; a checkout under `projects/` that the configuration does not declare writes the drift
  notice to stderr.
- `cargo dev gate slice` exits 0.
