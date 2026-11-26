---
name: ws-destroy-command
depends-on:
  - env-name-rule
  - env-selection
  - teardown-preview
---

# The `ws destroy` command

Adds `env_destroy::select` and the `ws destroy PATTERNS... [--force] [--dry-run] [--json]` command of
[destroy.md](../spec/destroy.md), Selection: the env-level pattern shape, the name rule of `env-name-rule`, the registry
read, `reach` with `EnvRule::Named` of `env-selection`, the no-match line, the confirmation, and every selected
environment torn down one at a time. Its row is that of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), Commands; the
prompt is the confirmation of [runtime.md](../../../../delivered/parallel-workspaces/component-runtime/spec/runtime.md).
Delivers the selection bullets of `product:destroy-environments` and `product:environment-targets` and
`product:environment-recovery` for teardown, under `decision:parallel-workspaces/destructive-approval`,
`decision:parallel-workspaces/partial-execution`, and `decision:parallel-workspaces/target-selection`; the serial order
is `engineering:bounded-fan-out`'s.

## Change

- `env_destroy::select` runs Selection steps 3 to 6: the name rule on each literal, the registry read, `reach`, and the
  rule on each glob match.
- `crates/core/src/commands/ws/destroy.rs` registers `ws destroy` with `--force`, `--dry-run`, and `--json`; it prompts
  when more than one PATTERN is given or any is a glob, unless `--force` or `--dry-run`, then calls `destroy` for each
  name in order, every one attempted, and exits 1 when any failed.
- `crates/core/topics/ws/destroy.md`, referenced from the `ws` group topic.

## Acceptance

- Component tests through `InProcess`: `ws destroy .`, `..`, `projects`, and `a/b` each exit 1 before any prompt, git
  process, or write.
- A component test: a literal naming no directory and no record exits 1 with `UnknownEnv` and nothing done; a literal
  whose directory is gone and whose record remains releases it and exits 0.
- A component test: `ws destroy beta 'a*'` destroys `beta`, then the present `a*` matches in name order, after one
  confirmation; answering no, or ending input, exits 1 with nothing done.
- A component test: a glob matching nothing writes "No environments matched: <patterns>" on stdout, under `--json` too,
  and exits 0.
- A component test: `--force` and `--dry-run` each skip the prompt; one literal alone never prompts.
- A component test: with two selected environments, the first failing, the second is still torn down, both have their
  `target_started` and `target_completed` events, and the command exits 1.
- A component test: `--json` writes shape C events alone on stdout.
- `cargo dev gate slice` exits 0.

## Out of scope

- `--strict` and `--no-provision-teardown` are registered by the epics that own the phases they read:
  `epic:running-environments/resource-provisioning` and `epic:agent-ready-operation/executable-extensions`.
