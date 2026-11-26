---
name: ws-init-all
depends-on:
  - allocation-rule
  - ws-init-env
---

# `ws init --all` reconciles every environment found

Adds `env_create::discover` and the `--all` option of `ws init` of [init.md](../spec/init.md), Running `ws init --all`
and Discovery for `--all`: the workspace part, the registry findings of [allocation.md](../spec/allocation.md),
Findings, as warnings once, then `reconcile_env` for each environment found from the first project's worktree
registrations. Delivers the discovery bullets of `product:environment-targets` and the no-deletion bullet of
`product:environment-state-recovery`, under `decision:parallel-workspaces/environment-lifecycle/discovery-authority`,
whose not-recreated registration is a registered deviation.

## Change

- `crates/core/src/env_create.rs` gains `discover`: the first project entry's `worktree_list`, each registration's
  parent directory directly under the root, a name whose directory is gone or that fails `env_name::check` skipped with
  a warning, the rest sorted without repeats.
- `crates/core/src/commands/ws/init.rs` registers `--all`, refused with `TARGET` as command-surface.md gives; the order
  is the load, the registry read, `bootstrap::reconcile`, the findings, `discover`, then every name attempted.
- `crates/core/topics/ws/init.md` describes `--all`.

## Acceptance

- A component test through `InProcess`: two environments created earlier are both found and reconciled after the
  workspace part, in name order; the command exits 0.
- A component test: a registration whose env directory was deleted is warned as not recreated, and no directory or
  worktree is created for it.
- A component test: a registration under a directory named `projects` or `.x` is skipped with a warning naming the
  fault.
- A component test: with the first project's source checkout absent, nothing is discovered and the workspace part still
  runs.
- A component test: a registry holding a shared index and an out-of-band index writes one warning each, once, before the
  first environment's events, and exit stays 0.
- A component test: one environment that fails does not stop a later one, and the command exits 1; so does a failing
  workspace part with every environment succeeding.
- A component test: an environment with its worktrees on disk and no registry record is reconciled and recorded, and no
  branch, worktree, or directory of it is removed.
- `cargo dev gate slice` exits 0.

## Out of scope

- `on_workspace_reconcile` in the workspace part belongs to `epic:agent-ready-operation/executable-extensions`.
