---
name: restack-results
depends-on:
  - restack-plan
  - restack-execute
---

# The restack documents and text

Adds the rendering of [restack-plan.md](../spec/restack-plan.md), Result, and
[restack-execution.md](../spec/restack-execution.md), Result: the refused, dry-run, and run documents in shape A of
[output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md), with no
discriminator field, and their text, each refusal with how to proceed. Delivers the preview bullets of
`product:declare-stack-order` and `product:review-replay-boundaries`, the reporting bullets of
`product:resume-stack-rewrite`, and `product:preserve-unrelated-work`'s refusal guidance, under
`decision:functional-parity` for wording.

## Change

- `crates/core/src/restack.rs` gains the refused and dry-run documents and text; `restack/execute.rs` gains the run
  document and text, with each link repository's `outcome` derived from the report.
- The run document lists every link of the plan bottom first, and `remaining` holds plan-shaped links and an empty
  `refusals`.
- No command calls them yet.

## Acceptance

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
- `cargo dev gate slice` exits 0.
