---
name: ws-destroy-command
depends-on:
  - env-name-rule
  - env-selection
  - teardown-preview
---

# The `ws destroy` command

Adds `env_destroy::run`, `select`, and the `ws destroy PATTERNS... [--force] [--dry-run] [--json]` command of
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

- `env_destroy.rs` gains `run`, `Destroyed`, `DestroyError`, and `select`, as destroy.md, Domain, gives them: `run`
  takes the start path and runs Selection steps 1 to 8, root discovery and the load, the pattern shape, the name rule on
  each literal, the registry read, and `select` (`reach` and the rule on each glob match); it calls the `confirm`
  closure when more than one PATTERN is given or any is a glob, unless `--force` or `--dry-run`, then `destroy` for each
  name in order, every one attempted, and answers `RunEnd` `Completed` with `success` false when any failed; when no
  name is selected it answers `Destroyed::NoMatch` carrying the no-match line it built.
- `crates/core/src/commands/ws/destroy.rs` registers `ws destroy` with `--force`, `--dry-run`, and `--json`, builds the
  `confirm` closure over `confirm.rs`, writing the "This will destroy" line, and the event sink, makes its one call to
  `run`, maps each `DestroyError` variant to its `Termination`, writes the line `Destroyed::NoMatch` carries, holding no
  literal of its own, and exits 1 when the run did not succeed.
- `crates/core/topics/ws/destroy.md`, referenced from the `ws` group topic.

## Acceptance

- Component tests through `InProcess`: `ws destroy .`, `..`, `projects`, and `a/b` each exit 1 before any prompt, git
  process, or write.
- A component test: a literal naming no directory and no record exits 1 with `UnknownEnv` and nothing done; a literal
  whose directory is gone and whose record remains releases it and exits 0.
- A component test: `ws destroy beta 'a*'` destroys `beta`, then the present `a*` matches in name order, after one
  confirmation; answering no, or ending input, exits 1 with nothing done.
- A unit test: `run` over globs matching nothing answers `Destroyed::NoMatch` whose line is "No environments matched:
  <patterns joined by ' '>".
- A component test: a glob matching nothing writes "No environments matched: <patterns>" on stdout, under `--json` too,
  and exits 0.
- A component test: `--force` and `--dry-run` each skip the prompt; one literal alone never prompts.
- A component test: with two selected environments, the first failing, the second is still torn down, both have their
  `target_started` and `target_completed` events, and the command exits 1.
- A component test: `--json` writes shape C events alone on stdout.
- `cargo dev gate slice` exits 0.

## Out of scope

- `--strict` and `--no-provision-teardown`, and their `Options` fields, are registered by the owners
  [destroy.md](../spec/destroy.md), System, names.
