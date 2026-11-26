---
name: module-scaffold
code:
  - winter-workspace/crates/core/src/lib.rs
  - winter-workspace/crates/core/src/termination.rs::Termination
  - winter-workspace/crates/core/src/termination.rs::CommandError
  - winter-workspace/crates/core/src/termination.rs::SelectionError
  - winter-workspace/crates/core/src/termination.rs::GitOperation
---

# The coordinated Git modules and error variants

Lands, in one change, the shared edits `context/architecture/changes.md` requires first of work built as parallel
changes: the module files present and empty, and the error variants the later slices render. These are the modules of
[ref-tokens.md](../spec/ref-tokens.md), [fan-out.md](../spec/fan-out.md),
[target-patterns.md](../spec/target-patterns.md), [integration.md](../spec/integration.md),
[tracking.md](../spec/tracking.md), [refresh.md](../spec/refresh.md), [merge.md](../spec/merge.md),
[push.md](../spec/push.md), [status.md](../spec/status.md), and [diff.md](../spec/diff.md), Domain, with the
`termination.rs` variants each lists. Shaped by `engineering:typed-outcomes`.

## Change

- `lib.rs` declares `ref_tokens`, `fan_out`, `targets`, `integration`, `tracking`, `refresh`, `merge`, `push`, `status`,
  and `diff`, each an empty file.
- `termination.rs` holds `RefTokenError` and `CommandError` gains `RefToken(#[from] RefTokenError)`; `SelectionError`
  gains `NotEnvRepo`; `Termination` gains `StatusCollection`, exit 2, as
  [results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md)
  defines it.
- `GitOperation` gains `UpstreamProbe`, `UnsetUpstream`, `SourceSync`, `Integrate`, `Abort`, `Push`, `TrackingRead`,
  `TrackingAheadProbe`, `LocalFastForward`, `LocalWorkProbe`, `TrackingAheadBehindProbe`, `Diff`, and
  `BranchAheadProbe`, each rendered with the name its leaf gives.
- Nothing constructs the new variants yet; no command changes.

## Acceptance

- Unit tests of the rendering: `RefTokenError::Unknown` reads "unknown ref token '{<token>}' in '<ref>'; accepted
  aliases: {main}, {master}, {default}"; `NoMainBranch` names the repository, the token, and the ref; `NotEnvRepo` reads
  "invalid pattern '<p>': expected <env>/<repo> (one '/' max)"; each new `GitOperation` renders its leaf's name.
- Unit tests: `Termination::Command` of `CommandError::RefToken` has exit 1, and `Termination::StatusCollection` exit 2.
- `cargo dev gate slice` exits 0.
