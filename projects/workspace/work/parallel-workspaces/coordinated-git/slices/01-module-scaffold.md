---
name: module-scaffold
code:
  - winter-workspace/crates/core/src/termination.rs::Termination
  - winter-workspace/crates/core/src/termination.rs::CommandError
  - winter-workspace/crates/core/src/termination.rs::SelectionError
  - winter-workspace/crates/core/src/termination.rs::GitOperation
  - winter-workspace/crates/dev/src/architecture.rs::COMMAND_FILE_NAMES
---

# The coordinated Git modules and error variants

Lands, in one change, the shared edits `context/architecture/changes.md` requires first of work built as parallel
changes: the module files present and empty, the error variants the later slices render, the names no command file may
hold, and the visibility the entry tests need. These are the modules of [ref-tokens.md](../spec/ref-tokens.md),
[fan-out.md](../spec/fan-out.md), [target-patterns.md](../spec/target-patterns.md),
[integration.md](../spec/integration.md), [tracking.md](../spec/tracking.md), [refresh.md](../spec/refresh.md),
[merge.md](../spec/merge.md), [push.md](../spec/push.md), [status.md](../spec/status.md), and
[diff.md](../spec/diff.md), Domain, with the `termination.rs` variants each lists, and `RefTokenError` of ref-tokens.md.
Shaped by `engineering:typed-outcomes`.

## Change

- `lib.rs` declares `ref_tokens`, `fan_out`, `targets`, `integration`, `tracking`, `refresh`, `merge`, `push`, `status`,
  and `diff`, each an empty file but `ref_tokens.rs`, which holds `RefTokenError` alone.
- `CommandError` gains `RefToken(#[from] RefTokenError)`, and `lib.rs` re-exports `RefTokenError` by name;
  `SelectionError` gains `NotEnvRepo`; `Termination` gains `StatusCollection`, exit 2, as
  [results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md)
  defines it.
- `GitOperation` gains `UpstreamProbe`, `UnsetUpstream`, `SourceSync`, `Integrate`, `Abort`, `Push`, `TrackingRead`,
  `TrackingAheadProbe`, `LocalFastForward`, `LocalWorkProbe`, `TrackingAheadBehindProbe`, `Diff`, and
  `BranchAheadProbe`, each rendered with the name its leaf gives.
- `fan_out` and `integration` join `COMMAND_FILE_NAMES` in `crates/dev/src/architecture.rs`, its pinned test renamed for
  the new count, since only domain entries call them (fan-out.md and integration.md, Domain); `ref_tokens`, `targets`,
  `tracking`, `refresh`, `merge`, `push`, `status`, and `diff` hold what a command file calls and do not join it.
- `context/architecture/structure.md` gains the rows of the ten modules and what each may call; `changes.md` gains the
  rows its tables require for each, `fan_out` and `integration` in the names a command file may not hold.
- The entries of later slices are re-exported by those slices, as the visibility paragraph of each leaf's Domain lists.
- Nothing constructs the new variants yet; no command changes.

## Acceptance

- Unit tests of the rendering: `RefTokenError::Unknown` reads "unknown ref token '{<token>}' in '<ref>'; accepted
  aliases: {main}, {master}, {default}"; `NoMainBranch` names the repository, the token, and the ref; `NotEnvRepo` reads
  `"invalid pattern '<p>': expected <env>/<repo> (one '/' max)"`; each new `GitOperation` renders its leaf's name.
- Unit tests: `Termination::Command` of `CommandError::RefToken` has exit 1, and `Termination::StatusCollection` exit 2.
- A unit test of the command-file scan in `crates/dev`: a command file that names `fan_out` or `integration` is a
  violation naming the file, the line, and the token.
- `cargo dev gate slice` exits 0.
