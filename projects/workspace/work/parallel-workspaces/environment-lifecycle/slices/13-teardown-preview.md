---
name: teardown-preview
depends-on:
  - env-teardown
---

# The teardown preview

Adds the `dry_run` option of `env_destroy::destroy` of [destroy.md](../spec/destroy.md), Per target, step 5, and the
`--dry-run` branch of step 1: after the same inventory and dirty refusal, it lists the worktrees it would remove, the
env directory it would remove or the leftovers it would keep, and the exclude block it would strip, and stops. Delivers
the preview bullet of `product:destroy-environments` and the read-only half of
`engineering:teardown-evidence-completeness`, under
`decision:parallel-workspaces/environment-lifecycle/teardown-policy`, whose kept-leftover preview is a registered
deviation.

## Change

- `Options` gains `dry_run`; with it, step 5 runs in place of steps 9 to 13 and writes nothing.
- `bootstrap.rs`'s `Action` gains `WouldRemoveWorktree`, `WouldRemoveEnv`, `WouldKeepEnv`, and
  `WouldRemoveWorkspaceExclude`; a missing env directory with a record reports `env_missing` "index <i> would be
  released".
- A failure to locate the exclude file fails the preview.

## Acceptance

- A component test with real git: the preview of a clean environment names each worktree, the env directory, and the
  exclude block, and afterwards every file, worktree, branch, and the state file are as before; a recording program port
  shows no `git worktree remove` and no write.
- A component test: an environment holding a stray file previews `would_keep_env` naming it, not `would_remove_env`.
- A component test: a dirty worktree refuses the preview as it refuses the teardown, unless `--force` is given.
- A component test: a missing env directory with a record previews the release and leaves the record in place.
- A component test: an exclude file that cannot be located fails the preview with "workspace exclude — <cause>".
- `cargo dev gate slice` exits 0.

## Out of scope

- The child preview and the provision plan, which run first in the spec's step 5, belong to
  `epic:parallel-workspaces/nested-workspaces` and `epic:running-environments/resource-provisioning`.
