---
name: reference-scenarios
depends-on:
  - ws-reset
  - ws-clean
  - ws-checkout
---

# Recorded scenarios of `ws checkout`, `ws reset`, and `ws clean`

Records `ws checkout`, `ws reset`, and `ws clean` against the reference tool and replays them against the binary,
proving the flows of [checkout.md](../spec/checkout.md), [reset.md](../spec/reset.md), and [clean.md](../spec/clean.md)
end to end under `decision:functional-parity`; each registered deviation of this epic carries its `[deviation]` decision
and expected result. Every fixture is a disposable workspace with fixture remotes, as
`engineering:destructive-fixture-confinement` asks.

## Change

- Scenarios under `scenarios/ws-checkout/`, `scenarios/ws-reset/`, and `scenarios/ws-clean/`, recorded with
  `cargo dev record` from the reference tool, each with a fixture workspace whose projects are cloned from fixture
  remotes.
- No production code changes.

## Acceptance

- `ws-checkout` scenarios: a feature branch adopted across two projects; one project falling back to its main; a refused
  dirty worktree; `--new`; `{main}`.
- `ws-reset` scenarios: `--soft`, `--mixed`, and `--hard --force` across two worktrees; a refused ambiguous sha; a
  missing ref; `--dry-run`.
- `ws-clean` scenarios: `--dry-run`; `--force` with an ignored file kept; nothing to clean.
- Deviation scenarios, each with its decision and `expected/`: the previous commit of each moved worktree and a blocking
  untracked file refused under `decision:parallel-workspaces/safe-history/pointer-recovery`; a checkout and a reset
  stopped part-way, and a clean stopped part-way, with their failed and unattempted targets, under
  `decision:parallel-workspaces/partial-execution`; an absent worktree named in `missing` and an unknown literal
  environment under `decision:parallel-workspaces/target-selection`.
- `cargo dev gate full` exits 0.
