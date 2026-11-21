---
name: bounded-fetch
depends-on:
  - git-port
---

# `fetch` with a timeout and bounded retry

Adds `fetch` to `git.rs` with the Remote calls of [standalone-pins.md](../spec/standalone-pins.md): the timeout
`WINTER_GIT_TIMEOUT_S` sets, the bound that
[child-processes.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/child-processes.md) leaves to the
epics that fetch. Nothing calls it yet.

## Change

- The program port's `Call` gains `timeout: Option<Duration>`; every existing call, the catalog's in `locations.rs`
  included, passes `None` and behaves as before.
- `OsPrograms` starts no thread: it polls the child's stdout and stderr with poll(2) until both close or the deadline on
  the monotonic clock passes. On expiry it lists git's direct children with `ps --ppid`, sends SIGKILL to git and then
  to each child, ignoring a failed listing or kill, reads the streams to their end, and answers `Signal(9)`.
- `fetch` takes the invocation's `ProcessEnv` and the entry's `Diagnostics` beside the ports. It runs `fetch origin`
  with the timeout read from the snapshot on every call, writing the spec's warning record for an invalid or
  non-positive value (`inf` and `nan` are invalid), and retries a transient failure up to three attempts with the
  jittered backoff, writing the spec's retry notice as a warning record per retry. Any other failure is the call's
  `RepoError` at once.
- `programs.rs` supplies the sleep and the jitter draw, through `fastrand`, and the retry loop takes them, so a test
  fixes both. `context/standards/rust.md`, Output and environment access, names `programs.rs` as the one library file
  that reads the monotonic clock, sleeps, and draws a random number, as the exception to its "reads no clock".
- The architecture check forbids `Instant::now`, `thread::sleep`, `fastrand`, and `rustix` in the library outside
  `programs.rs`, with the pinned test of the token list updated and the four tokens added to the token table of
  `context/standards/rust.md`, each marked as allowed in `programs.rs` alone.
- The library gains `rustix` (`event`, `process`) and `fastrand`, listed in the architecture context's dependency table.
- `fetch` opens one `git fetch` span over all its attempts.

## Acceptance

- Component tests with a stub `git` first on the isolated tree's `PATH`: a stub that sleeps past
  `WINTER_GIT_TIMEOUT_S=0.2` is killed and retried; a stub that fails twice with
  `fatal: the remote end hung up
  unexpectedly` and then succeeds answers success after two notices in the
  `Diagnostics` buffer; a stub failing with `Permission denied (publickey)` fails after one attempt and no notice.
- Component tests of the wait: a stub that writes 1 MiB to stderr and exits 0 within the timeout succeeds with the whole
  stderr captured; a stub that starts a child holding stderr open and then sleeps past the timeout returns within a
  second of the deadline as `Signal(9)`, its child killed, and stderr the spec's timeout line.
- Unit tests of the timeout value: unset is 40; `abc`, `inf`, `nan`, and `-1` warn and are 40; `2.5` is 2.5.
- A unit test of the delay: attempts 1, 2, 3, 4, 5 with `u = 1` are 1.5, 3, 6, 8, 8 seconds, and with `u = -1` are 0.5,
  1, 2, 4, 4.
- A component test against the git fixtures: `fetch` brings a commit pushed to the remote, and leaves a local tag in
  place after the remote's tag was moved.
- A component test with tracing on: a fetch retried once records one `git fetch` span.
- A unit test of the architecture check: `thread::sleep` in a library file other than `programs.rs` fails it, and in
  `programs.rs` passes.
- `cargo dev gate slice` exits 0.
