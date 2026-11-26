---
name: ws-restack-command
depends-on:
  - restack-plan
  - restack-execute
---

# The `ws restack` command

Adds the `ws restack ENV... BASE [--cut TEXT] [--dry-run] [--json]` command: its row and combination rules, the literal
chain-name shape included, are those of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), checked by
`chain` of `restack-plan`; it loads the workspace, runs `plan`, then writes the refused or dry-run result, or runs
`execute` of `restack-execute` and writes its result. The command file builds the documents and text of
[restack-plan.md](../spec/restack-plan.md), Result, and [restack-execution.md](../spec/restack-execution.md), Result,
from the domain's values (`context/architecture/changes.md`, A command): shape A of
[output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md), with no
discriminator field, each refusal with how to proceed. Its exits are those of
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
and its topic is restack-plan.md, Help. Delivers the help bullet and the preview bullet of
`product:declare-stack-order`, the dry-run and preview bullets of `product:review-replay-boundaries`, the reporting
bullets of `product:resume-stack-rewrite`, and `product:preserve-unrelated-work`'s refusal guidance, under
`decision:command-vocabulary` and `decision:functional-parity` for wording; it runs one repository at a time, as
[fan-out.md](../../coordinated-git/spec/fan-out.md), Bounds, lists.

## Change

- `crates/core/src/commands/ws/restack.rs` registers the command; a `ChainError` of `chain` ends as
  `CommandError::Chain`, exit 1, before any git call; `plan`'s `RestackError` ends as restack-plan.md, Domain, maps it;
  a refused plan writes its result and ends `Reported`, exit 1; `--dry-run` writes the plan and exits 0 with no write
  call; a run exits 0 when complete and `Reported`, exit 1, after a conflict or failure.
- The command file builds the refused, dry-run, and run documents and their text; each link repository's `outcome` in
  the run document is derived from the report; the run document lists every link of the plan bottom first, and
  `remaining` holds plan-shaped links and an empty `refusals`.
- `--json` writes one document on stdout, at the end.
- `crates/core/topics/ws/restack.md`, referenced from the `ws` group topic, states the chain order, participation,
  `--cut`'s scope, and how to resume, as Help gives.

## Acceptance

- Component tests through `InProcess`: `ws restack 'a*' main`, `ws restack a/b main`, an empty ENV, and an empty BASE
  each exit 1 with the `ChainError` the spec gives on stderr, nothing on stdout, and no git call, as a recording program
  port shows.
- Component tests: `ws restack` with no token exits 2 with the parser's usage error; `ws restack main` exits 1 with
  `OneToken` and no git call.
- Unit tests of the refused document: each `Refusal` serialises to its kebab-case `result`, with `repos` and `element`
  as the spec gives, `element` `null` where there is none.
- Unit tests of the refused text: each refusal kind prints its result, `<env> onto <predecessor>`, its repositories, its
  element, and its hint from the spec's table.
- Unit tests of the dry-run document and text: `boundary` and `source` are `null` and `-` for a skipped repository.
- Unit tests of the run document: `outcome` is `rebased`, `up-to-date`, or `conflict` for a repository reached,
  `skipped` for a non-participating one, and `null` for the failed repository and for one never reached; `conflict` and
  `failure` are `null` when absent; `remaining` is empty after a complete run.
- Unit tests of the run text: a conflict names the replayed commit, the link, the repository, and each path, then
  `git rebase --continue`, `git rebase --abort`, and re-running with the same `--cut`; a failure names its repository
  and message; a complete run says so.
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
