---
name: update-command
depends-on:
  - bounded-fetch
  - resolve-and-materialize
  - epic:parallel-workspaces/repository-catalog
---

# `ws update` re-pins the selected standalones

Adds `ws update [REPOS]... [--autostash] [--json]` running the Re-pin flow, with the `ws update` events and text of
[standalone-pins.md](../spec/standalone-pins.md) and the drift notice of
[drift-warning.md](../../repository-catalog/spec/drift-warning.md). Its arguments and bare-name refusal are those of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), and its
events shape C of
[output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md). Delivers
`product:update-standalone-inputs`, the resolution-order and fetch bullets of `product:standalone-reference-policy`, and
`engineering:repin-preservation-lifetime`, under `decision:parallel-workspaces/reproducible-inputs/update-preservation`.

## Change

- The command file registers the command with the arguments and bare-name shape command-surface.md gives it, without
  `--freeze` and `--force`. It builds `Diagnostics` from `cx.err` and `cx.log` and hands `pins::repin` the file ports,
  `cx.programs`, `cx.tracer`, `cx.env`, and the `Diagnostics`, then maps `PinError` to its `Termination`.
- `pins/repin.rs` writes the drift notice through `Diagnostics` before selection, from `drift::detect` and
  `drift::notice_line`.
- Selection reads a pattern holding `*`, `?`, or `[` as a case-sensitive `fnmatchcase` glob and any other as a literal;
  it refuses an unpinned or unknown literal as `RepoMessage::NotPinned` or `NoPinnedNamed`, keeps every pinned
  standalone a glob matches, and drops a missing checkout, writing the spec's `warning:` line through `Diagnostics`.
- Each selected standalone is fetched, guarded, optionally stashed, resolved, and either reported up to date or checked
  out and upserted; the pop runs on every path after a stash, and a failed pop is a warning record naming the checkout.
- A failed head read or a lock error ends the run as that `PinError` after the pop, with no `pull_completed`.
- Events are written and flushed as they happen; the run exits 1 when an outcome is `pin_error`.
- What a lock entry whose field is not a string reads as is settled by the recorded scenario below, the probe
  lock-and-state.md, Open questions, hands this epic; the reader follows the record, and the delivered spec's open
  question is answered from it in a change to the planning repository.

## Acceptance

- Component tests of re-pinning: a declaration changed to a newer tag reports `re_pinned` with the new commit's first 8
  characters and rewrites only that entry; a second run reports `up_to_date` and leaves the lock's bytes unchanged; a
  tag moved on the remote reports `up_to_date`; an annotated tag reports `re_pinned` on every run.
- A component test: a branch pin with a local-only commit is re-pinned to `origin/<ref>` and reports `re_pinned`.
- Component tests of preservation, each with `--autostash` and a modified tracked file: after a successful checkout,
  after an unresolvable ref, and after a failed checkout, the modification is restored; after a pop conflict the outcome
  stays `re_pinned`, a warning names the checkout, and the stash entry remains. Without `--autostash` a dirty checkout
  is `pin_error` and exits 1.
- Component tests: an unresolvable ref is `pin_error` while the next standalone still re-pins; a failed fetch is logged
  and resolution uses the refs on disk; a re-pin whose checkout git refuses is `pin_error` with the lock's bytes
  unchanged; an unresolvable ref's `--json` `pin_ref` is the rendered error, its message then its `cwd:` line.
- A component test: a malformed `.winter/config.lock` ends the run as the configuration error, exit 1, with no
  `pull_completed`, every checkout's HEAD and the lock's bytes unchanged.
- Component tests of selection: an unknown literal and an unpinned literal are the reference's errors, exit 1; a glob
  matching none and a run with every selected checkout missing write `Nothing to update` and exit 0; two literals select
  exactly those two; `lib-[ab]` selects `lib-a` and `lib-b` and not `Lib-a`; an empty name and `a/b` are the command
  surface's bare-name refusals.
- Component tests of the `up_to_date`, `re_pinned`, and `pin_error` rows and both `pull_completed` rows, in text and
  `--json`; the drift notice precedes `→ pulling` when a declared project is missing on disk.
- A recorded scenario `scenarios/ws-update/lock-field-not-string`, recorded with `cargo dev record` from the reference
  tool with a lock entry whose `commit` is an integer, passes `cargo scenarios ws-update`.
- `cargo dev gate slice` exits 0.

## Out of scope

- `--freeze` and `--force`, and the `pinned`, `already_pinned`, and `refused` rows: `freeze`.
