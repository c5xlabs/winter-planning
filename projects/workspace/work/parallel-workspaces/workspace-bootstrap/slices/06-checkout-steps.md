---
name: checkout-steps
depends-on:
  - module-scaffold
  - port-extensions
  - epic:parallel-workspaces/reproducible-inputs
---

# One repository's reconciliation steps

Adds `checkout::run` of [reconciliation.md](../spec/reconciliation.md), Interfaces, One repository and Commands: the
location step with clone, verification, and clone confinement, then identity, per-repository excludes, `cmd`, and the
pin through `materialize` of `epic:parallel-workspaces/reproducible-inputs`; with the git operations `clone`,
`toplevel`, `origin_url`, and `set_config`. Delivers `product:reconcile-repositories` and
`product:recorded-input-materialization` for one repository, under
`decision:parallel-workspaces/workspace-bootstrap/reconciliation-scope` and
`decision:parallel-workspaces/workspace-bootstrap/clone-confinement`.

## Change

- `crates/core/src/git.rs` gains `clone`, `toplevel`, `origin_url`, and `set_config`, each with its operation span.
- `crates/core/src/bootstrap/checkout.rs` holds `run`, `confined`, and `exclude_path`; it emits `InitEvent`s through a
  callback and returns the first failure, so the steps after it do not run.
- `cmd` runs through `Program::Shell` with `stream`, the environment additions, and, for a nested project, the removals
  the spec lists.
- Nothing outside tests calls `run` yet.

## Acceptance

- Component tests with real git and a fixture remote: an absent location is cloned and reported `cloned`, a project on
  its `main_branch` when the remote's default HEAD differs; a present clone of the declared URL is `exists`, a clone of
  an empty remote included; a present directory that is not a work tree top and one whose `origin` differs each fail
  with their `RepoMessage` and run no later step.
- Component tests: a standalone whose path is the root, resolves through a symbolic link outside the root, lies under
  `.winter` or `projects`, lies inside another declared standalone, or contains one fails with `UnsafeLocation` and
  creates nothing.
- Component tests: identity is written to the clone's own config when both name and email exist; excludes append
  `# winter-managed` and only the entries not already present, and a second run appends nothing; a worktree checkout's
  entries go to the common `info/exclude`.
- Component tests: each `cmd` runs in the location with `CI=1` and `NO_COLOR=1` set, its lines arrive as `CmdOutputLine`
  events, a non-zero exit fails with `CommandFailed` and skips the remaining commands and the pin; a nested project's
  command does not see `WINTER_*` other than the kept ones; `LC_ALL` and `LANGUAGE` reach the command as the snapshot
  holds them, and `WINTER_LAUNCH_TIME` does not.
- Component tests: a standalone with a fresh tag record is checked out at the recorded commit and reported `pinned`; one
  with a fresh branch record is reported `pinned` at its branch; a dirty checkout with a stale record fails with the
  pin's refusal.
- Component tests: a malformed lock fails the standalone with the lock's `ConfigError` text and writes no record; with
  two standalones, one failing its pin leaves the other's record as it was.
- A unit test: the failure text names the failed step and the steps not attempted.
- `cargo dev gate slice` exits 0.
