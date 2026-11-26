---
name: teardown-audit
depends-on:
  - status-read
  - epic:parallel-workspaces/environment-lifecycle
---

# The child audit and loss refusal of `ws destroy`

Adds `Child`, `Audit`, `audit`, and `losses` of [teardown.md](../spec/teardown.md), Phases, phases 0 and 1, at steps 3
and 4 of [destroy.md](../../environment-lifecycle/spec/destroy.md), Per target, which
`epic:parallel-workspaces/environment-lifecycle` builds: every child of the env read through `Nesting::state` of
`status-read` before anything is torn down, an unreadable child refusing the target whatever `--force`, and the child's
dirty envs and unpushed entries added to the env's own refusal lists without `--force`. Delivers the audit and refusal
bullets of `product:nested-teardown-preservation`, the acceptance of `engineering:child-authority-confinement`, and
`engineering:nested-loss-evidence` for the refusal, under
`decision:parallel-workspaces/nested-workspaces/recursive-teardown`, whose phase 0 under `--force` is a registered
deviation.

## Change

- `crates/core/src/nested/teardown.rs` holds `Child`, `Audit`, `audit`, and `losses`; `audit` reads every child in
  configuration order, emitting "nested workspace state — <error>" for each unreadable one and reading the rest.
- `env_destroy::destroy` holds one `Nesting` for the invocation, runs `audit` at step 3 over the declared worktrees of
  nested projects, and adds `losses` to step 4's lists; the unpushed entries refuse with the refusal teardown.md gives.
- `--dry-run` refuses on the audit and losses as the teardown does.

## Acceptance

- Component tests with real git and a `winter` stand-in: an env whose nested child reports a dirty env refuses with
  `Dirty` naming "<repo> (nested: <env>)", and one whose child holds an unpushed branch refuses naming "<repo> (nested:
  <env>/<repo> (unpushed branch))" and saying the work would be deleted with the worktree; in each, nothing is removed
  and no child `ws destroy` starts; with `--force` neither refusal is made.
- A component test: a child whose document leaves out `stashes` refuses as unpushed with "stashes unknown".
- A component test: with `--force`, a child whose status reports another root fails the target with "refusing to destroy
  — cannot read nested workspace: <repo>; keeping the env.", and the stand-in records no command but the status read;
  the worktree and the env directory stay.
- A component test: of two nested children, the first unreadable, the second is still read, and the target refuses
  naming the first alone.
- A component test: the audit's status reads all run before any child `ws destroy` is started, as a recording program
  port shows.
- `cargo dev gate slice` exits 0.

## Out of scope

- The preview and the child teardown land with `teardown-preview` and `child-retirement`.
