---
name: ws-restack-command
depends-on:
  - restack-plan
  - restack-execute
  - restack-results
---

# The `ws restack` command

Adds the `ws restack ENV... BASE [--cut TEXT] [--dry-run] [--json]` command: its row and combination rules, the literal
chain-name shape included, are those of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md); it loads the
workspace, runs `plan` of `restack-plan`, then writes the refused or dry-run result, or runs `execute` of
`restack-execute` and writes its result, through `restack-results`. Its exits are those of
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
and its topic is [restack-plan.md](../spec/restack-plan.md), Help. Delivers the help bullet of
`product:declare-stack-order` and the dry-run bullet of `product:review-replay-boundaries`, under
`decision:command-vocabulary`; it runs one repository at a time, as [fan-out.md](../../coordinated-git/spec/fan-out.md),
Bounds, lists.

## Change

- `crates/core/src/commands/ws/restack.rs` registers the command; a refused plan writes its result and ends `Reported`,
  exit 1; `--dry-run` writes the plan and exits 0 with no write call; a run exits 0 when complete and `Reported`, exit
  1, after a conflict or failure.
- `--json` writes one document on stdout, at the end.
- `crates/core/topics/ws/restack.md`, referenced from the `ws` group topic, states the chain order, participation,
  `--cut`'s scope, and how to resume, as Help gives.

## Acceptance

- Component tests through `InProcess`: `ws restack 'a*' main`, `ws restack a/b main`, an empty ENV, and an empty BASE
  each exit 2 before any git call.
- A component test: `ws restack upper lower main --dry-run --json` writes the plan document, exits 0, and leaves every
  ref, index, and worktree unchanged, as a recording program port running no `git rebase` shows.
- A component test: a dirty worktree in the chain exits 1 with the refused document and nothing moved; in text, the
  refusal's hint follows it.
- A component test: a conflicting chain exits 1 with the run document's `conflict`; re-running the same command after
  `git rebase --continue` completes and exits 0.
- A component test: `ws restack upper lower '{mian}'` exits 1 with the ref-token command error and nothing on stdout.
- Component tests: `ws restack --help` shows `ENV... BASE` and its options, and `context ws/restack` writes the topic's
  chain order, participation rule, and resume step.
- `cargo dev gate slice` exits 0.
