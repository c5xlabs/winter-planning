---
name: teardown-preview
depends-on:
  - teardown-audit
  - mutating-calls
---

# The child preview of `ws destroy --dry-run`

Adds `preview` of [teardown.md](../spec/teardown.md), Phases, `--dry-run`, run first in step 5 of
[destroy.md](../../environment-lifecycle/spec/destroy.md), Per target: after the audit and losses of `teardown-audit`,
each child env that would be destroyed and each child whose workspace services would be stopped, judged by
`binds_service` of `mutating-calls`, with no child call beyond the audit's status reads. Delivers the preview bullet of
`product:nested-teardown-preservation`.

## Change

- `crates/core/src/nested/teardown.rs` gains `preview`; `bootstrap.rs`'s `Action` gains `WouldDestroyNestedEnv` and
  `WouldStopNestedServices`, with their text.
- `env_destroy::destroy` runs `preview` before the env's own preview; a `binds_service` failure fails the preview with
  "nested workspace services — <error>".

## Acceptance

- A component test with real git and a `winter` stand-in: `ws destroy alpha --dry-run` over a child of two envs that
  binds `service = "tmux"` emits `would_destroy_nested_env` for each env at `<root>/<env>`, then
  `would_stop_nested_workspace_services` at the child root, before `would_remove_worktree` of the outer worktree; the
  stand-in records the status read alone, and every file is as before.
- A component test: a child binding no service previews no service stop and the preview succeeds.
- A component test: a child whose `config.local.toml` is not TOML fails the preview with "nested workspace services —"
  and its cause.
- `cargo dev gate slice` exits 0.
