---
name: fan-out
depends-on:
  - module-scaffold
---

# The bounded fan-out

Adds `fan_out::fan_out`, `Group`, `Message`, and the bounds of [fan-out.md](../spec/fan-out.md): groups of tasks run at
most `bound` at once on `std::thread::scope` workers, one group's tasks in order, with each worker's diagnostic lines
and results handed to the calling thread as they happen. This is the contract `ws fetch`, `ws pull`, `ws merge`, and
`ws status` run on here, and the nested status reads of `epic:parallel-workspaces/nested-workspaces` after this epic.
Delivers `engineering:bounded-fan-out` for those commands and the failure isolation of `engineering:shared-git-effects`.

## Change

- `crates/core/src/fan_out.rs` holds `REMOTE_TASKS`, `STATUS_PROBES`, `Group`, `Message`, and `fan_out`; each worker
  owns a `Diagnostics` writing into a buffer that sends each complete line as `Message::Line`.
- Results come back in input order, whatever the completion order; an `Err` from `on_message` lets running tasks finish,
  starts no new one, and returns that `Termination`; a worker panic is `InternalError::Panic` after the others finish.
- No command calls it yet.

## Acceptance

- A unit test over a recording `RunPrograms` with twenty groups: never more than `bound` git calls in flight, and the
  tasks of one group never overlap.
- A unit test: with tasks finishing in reverse order, `on_message` sees `Done` in completion order and `fan_out` answers
  the results in input order.
- A unit test: a line a worker writes reaches `on_message` before that task's `Done`, and two workers' lines never
  interleave within a line.
- A unit test: one task's failure result leaves every other task's result in place.
- A unit test: an `Err` from `on_message` on the first `Done` starts no further group and is the answer.
- A unit test: a panicking task ends `InternalError::Panic` after the other groups complete.
- `cargo dev gate slice` exits 0.
