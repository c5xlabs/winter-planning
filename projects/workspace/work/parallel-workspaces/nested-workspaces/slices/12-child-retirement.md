---
name: child-retirement
depends-on:
  - teardown-audit
  - mutating-calls
---

# Retiring each child before the outer teardown

Adds `Forwarded`, `retire_envs`, and `stop_services` of [teardown.md](../spec/teardown.md), Phases, phases 2 and 3, and
Forwarded, at step 6 of [destroy.md](../../environment-lifecycle/spec/destroy.md), Per target: each child env destroyed
by the child's own `winter ws destroy <env>` with the forwarded options, every one attempted, then each child that binds
a service stopped by `winter service down workspace`, all through `run` of `mutating-calls` and before any outer
teardown effect. A failure in either keeps the outer env whatever `--force`. Delivers the retirement, ordering, and
failure bullets of `product:nested-teardown-preservation`, under
`decision:parallel-workspaces/nested-workspaces/recursive-teardown`, whose phases 2 and 3 under `--force` are registered
deviations; children run serially as `engineering:bounded-fan-out` binds a command that removes files.

## Change

- `crates/core/src/nested/teardown.rs` gains `Forwarded`, `retire_envs`, and `stop_services`; `bootstrap.rs`'s `Action`
  gains `NestedEnvDestroyed` and `NestedServicesStopped`, with their text.
- `env_destroy::destroy` builds `Forwarded` from `Options` as teardown.md gives, with `strict` false and
  `provision_teardown` true while `Options` has neither field, and runs step 6 before step 7.
- A phase 2 failure emits its abort on the env and runs neither phase 3 nor any later step; a phase 3 failure does the
  same for the steps after it.
- `crates/core/topics/ws/destroy.md` says that a nested workspace's envs and workspace services are retired first and
  that a failure keeps the env even under `--force`.

## Acceptance

- A component test with real git and a `winter` stand-in: destroying an env whose child holds `a` and `b` and binds a
  service runs `winter ws destroy a`, `winter ws destroy b`, then `winter service down workspace` in the child, each
  between `CmdStarted` and `CmdCompleted`, then removes the outer worktree; `nested_env_destroyed` and
  `nested_workspace_services_stopped` are reported.
- Unit tests of the arguments: `Forwarded { force: true, strict: true, provision_teardown: false }` appends `--force`,
  `--strict`, and `--no-provision-teardown`, and the default appends none.
- A component test: with `--force`, a child `ws destroy a` exiting 1 still attempts `b`, then fails the target with
  "aborting destroy — a nested env could not be destroyed; keeping the env.", runs no service stop, and removes no
  worktree.
- A component test: with `--force`, a failing `service down workspace` fails the target with its abort, and the outer
  worktree, env directory, exclude block, and index stay.
- A component test: a child that binds no service runs no `service down`, and the target succeeds.
- A unit test over a recording program port: one child call at a time, and every child call ends before the first
  `git worktree remove`.
- `cargo dev gate slice` exits 0.

## Out of scope

- Registering `--strict` and `--no-provision-teardown` on `ws destroy` belongs to
  `epic:running-environments/resource-provisioning` and `epic:agent-ready-operation/executable-extensions`, which set
  `Forwarded` from the fields they add.
