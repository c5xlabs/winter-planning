# Merge

The `ws merge` command: integrating one typed ref into selected worktrees and standalones, on local refs only. Its
options and refusals, a PATTERN required unless `--standalone` included, are the `ws merge` row of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md); shape C is
that of [output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md), whose
event fields this leaf owns. Selection is [target-patterns.md](./target-patterns.md) with `EnvRule::Present`; the ref
expands as [ref-tokens.md](./ref-tokens.md); the parallelism is [fan-out.md](./fan-out.md); the integration is
[integration.md](./integration.md); a held pin is `resolve` of
[standalone-pins.md](../../../../delivered/parallel-workspaces/reproducible-inputs/spec/standalone-pins.md), Resolution.
The drift notice is that of
[drift-warning.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/drift-warning.md). Reference
locations are paths under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`; "WMS" is
`modules/workspace/workspace_merge_service.py`, "WRR" `modules/workspace/internal/write_repo_repository.py`. Message
wording is the binary's own (`decision:functional-parity`).

## Domain

```rust
// crates/core/src/merge.rs — product:integrate-selected-repositories,
// decision:parallel-workspaces/coordinated-git/integration-conflicts,
// decision:parallel-workspaces/coordinated-git/pinned-skips
pub(crate) enum PinnedScope { Include, Exclude, Only }   // default; --exclude-pinned; --only-pinned
pub(crate) enum MergeMode { FfOnly, Merge, NoFf }        // default; --merge; --no-ff -> Fallback None, Merge, NoFf

pub(crate) enum MergeEvent {
    Started { source_ref: String },                                       // merge_started, the ref as typed
    Merged { scope: String, repo: String, result: MergeResult, ahead: u64, behind: u64 },  // repo_merged
    Completed { success: bool },                                          // merge_completed
}
pub(crate) enum MergeResult { /* string values below */ }

pub(crate) fn merge(ports: &Ports<'_>, diagnostics: &mut Diagnostics<'_>, start: &Path, source_ref: &str,
    patterns: &[Pattern], scope: Scope, mode: MergeMode, autostash: bool, pinned: PinnedScope,
    sink: &mut dyn FnMut(&MergeEvent) -> Result<(), Termination>) -> Result<RunEnd, MergeError>;
pub(crate) enum MergeError {
    Config(Box<ConfigError>), Selection(SelectionError), RefToken(RefTokenError), Internal(Box<InternalError>),
}
```

## Interfaces

```text
# reference: modules/workspace/handlers/workspace_handler.py:753-776, WMS:74-276, WRR:121-163,879-966
order
  1. root discovery and the load; 2. the drift notice
  3. selection with PATTERNS (none when --standalone), then pinned scope: Include keeps every worktree, Exclude
     drops pinned ones, Only keeps only pinned ones; --standalone and --all reach every standalone
  4. no target -> no event; text "Nothing to merge", dim; --json writes nothing; exit 0
  5. the ref per target repository: ref_tokens::expand(source_ref, repo, main_branch), once per project and
     once per standalone, every one before the first git call; a refusal ends the command as
     CommandError::RefToken, exit 1, before merge_started
  6. merge_started; then, as fan-out.md REMOTE_TASKS, one group per project (its worktrees in selection order)
     and one per standalone; no fetch anywhere
  7. merge_completed { success }; exit 0 on success, else 1 (Termination::Reported)
per target
  standalone declaring a ref that resolves as Tag or Commit -> held_pin, nothing else runs
     # registered deviation, decision:parallel-workspaces/coordinated-git/pinned-skips: the reference merges into
     # it and moves HEAD off the locked commit (WMS:243-250 ignores pins); a ref that does not resolve is the
     # resolve error, result "error"; a branch pin is merged like any standalone, its lock left as it is
  git rev-parse --verify --quiet <expanded>^{commit} non-zero -> skipped-missing-ref
  otherwise integrate(path, expanded, mode's Fallback, autostash) of integration.md
  a GitFailure::Repo of the target -> result "error", its rendered error as a stderr line; the others go on
success: every result is fast_forwarded, up_to_date, merged, or held_pin
```

```json
{"type": "merge_started", "source_ref": "<as typed>"}
{"type": "repo_merged", "scope": "<env> | standalone", "repo": "<name>", "result": "<MergeResult>", "ahead": 0, "behind": 0}
{"type": "merge_completed", "success": true}
```

```text
result   the reference's fast_forwarded, up_to_date, merged, diverged, skipped-missing-ref; this epic's
         dirty_blocked, autostash_failed, abort_failed, held_pin, error. Integrated's Rebased never occurs here
ahead, behind   Diverged's counts, else 0
```

| Result                                                       | Stream    | Line                                             |
| ------------------------------------------------------------ | --------- | ------------------------------------------------ |
| `merge_started`                                              | stdout    | `→ merging <source_ref>`                         |
| `fast_forwarded`, `up_to_date`                               | stdout    | `[<scope>/<repo>] fast-forwarded`, `up-to-date`  |
| `merged`                                                     | stdout    | `[<scope>/<repo>] merged (merge commit created)` |
| `held_pin`                                                   | stdout    | `[standalone/<repo>] held @ <declared ref>`      |
| `diverged`                                                   | stderr    | `[<scope>/<repo>] diverged: +<ahead>/-<behind>`  |
| `skipped-missing-ref`                                        | stderr    | `[<scope>/<repo>] skipped: source ref not found` |
| `dirty_blocked`, `autostash_failed`, `abort_failed`, `error` | stderr    | `[<scope>/<repo>] <what happened>`               |
| `merge_completed`                                            | as pull's | `✓ merge complete`, `✗ merge had errors`         |

Reference rows: `modules/workspace/merge_reporter.py:47-75`. The sink is built as `ws pull`'s in
[refresh.md](./refresh.md).
