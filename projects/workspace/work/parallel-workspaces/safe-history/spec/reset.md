# Reset

The `ws reset` command: moving the pointer of each selected worktree to a ref without touching tracking or attachment.
Its options and combination rules are the `ws reset` rows of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md); its NDJSON
events, whose fields this leaf owns, are the `ws reset` vocabulary of
[output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md); the prompt is
the confirmation of [runtime.md](../../../../delivered/parallel-workspaces/component-runtime/spec/runtime.md),
Confirmation, under the `ws reset --hard` row of
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
Confirmation prompts. Selection is that of [target-patterns.md](../../coordinated-git/spec/target-patterns.md), tokens
expand as [ref-tokens.md](../../coordinated-git/spec/ref-tokens.md), and the command is serial, as
[fan-out.md](../../coordinated-git/spec/fan-out.md) lists. The probes and the previous position are those of
[safety-probes.md](./safety-probes.md); the drift warning is that of
[drift-warning.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/drift-warning.md). Reference
locations are paths under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`. Message wording is the
binary's own (`decision:functional-parity`).

## Domain

```rust
// crates/core/src/history/reset.rs — product:reset-worktree-pointers
pub(crate) enum ResetMode { Soft, Mixed, Hard }       // "soft" | "mixed" | "hard"; Mixed when none is given
pub(crate) enum ResetResult {
    Reset,                    // "reset": the plan under --dry-run, done otherwise
    RefusedAmbiguousSha,      // "refused-ambiguous-sha"   never bypassed
    RefusedMissingRef,        // "refused-missing-ref"     never bypassed
    RefusedDirty,             // "refused-dirty"           --hard only; bypassed by --force
    RefusedAbandonment,       // "refused-abandonment"     --hard only; bypassed by --force
    Failed,                   // "failed"                  registered deviation (Execution)
    Unattempted,              // "unattempted"             registered deviation (Execution)
}
pub(crate) enum ResetEvent<'r> {
    Started { r#ref: &'r str, mode: ResetMode, dry_run: bool,
              missing: &'r [(Selected, Missing)] },   // written as the target_missing events before reset_started
    Repo(&'r RepoReset),
    Completed { success: bool },
}
pub(crate) struct RepoReset {
    pub env: String, pub repo: String, pub result: ResetResult,
    pub r#ref: String,                // the expanded ref on reset, failed, and unattempted; "" on a refusal
    pub previous: Option<Sha>,        // registered deviation: the commit HEAD named before it moved
    pub error: Option<String>,        // failed: the repo error's message and git's stderr
}
// The command file's one domain call (context/architecture/changes.md, A command): Running, steps 1-8, from the start
// path, as the built locations::list and pins::repin take it. confirm is called with the targets exactly when step 5
// prompts; the command file builds it over confirm.rs (runtime.md, Confirmation), writing the preview line and asking
// "Continue?". sink receives each event as it is reached. An Err from either is the ending, answered as
// RunEnd::Stopped with nothing written to a worktree after it; Completed { success } otherwise
pub(crate) fn run(ports: &Ports<'_>, diagnostics: &mut Diagnostics<'_>, start: &Path, patterns: &[Pattern],
    r#ref: &str, mode: ResetMode, force: bool, dry_run: bool,
    confirm: &mut dyn FnMut(&[Selected]) -> Result<(), Termination>,
    sink: &mut dyn FnMut(&ResetEvent<'_>) -> Result<(), Termination>) -> Result<RunEnd, ResetError>;
// the failures that end the command before any effect; the command file maps: Config -> Termination::Config, EnvName
// -> Termination::Command(CommandError::EnvName), Selection -> Command(Selection), RefToken -> Command(RefToken),
// Repo -> Termination::Repo, Internal -> Termination::Internal, each box as it is
pub enum ResetError {
    Config(Box<ConfigError>),          // root discovery and the load
    EnvName(EnvNameError),             // target-patterns.md's name refusal
    Selection(SelectionError),         // target-patterns.md's selection refusals
    RefToken(RefTokenError),           // an unknown token (ref-tokens.md), before any git call
    Repo(Box<RepoError>),              // a probe's repo error (Preflight)
    Internal(Box<InternalError>),      // a git that cannot be started, or a candidate directory that cannot be listed
}
```

`ResetError` is `pub` and re-exported by name from `lib.rs` with the types its variants carry
(`context/architecture/changes.md`, An error, of `winter-workspace`); the other names here land with the command that
calls them and stay `pub(crate)`.

## Interfaces

### Running `ws reset PATTERNS... REF`

```text
# reference: modules/workspace/handlers/workspace_handler.py:493-559, env_reset_service.py:72-141
order
  0. the command file: the combination rules of command-surface.md, before the call
  steps 1-8 are run's
  1. root discovery and the load; a failure is ResetError::Config, exit 1
  2. the drift warning
  3. targets := the non-pinned worktrees PATTERNS select (target-patterns.md), in its order; missing ones warned
     about and skipped there; a refusal is ResetError::EnvName or Selection, exit 1
  4. no targets -> Nothing matched (Result); exit 0
  5. --hard, more than one target, neither --force nor --dry-run -> confirm(targets), whose closure writes the
     preview, "This will hard-reset <n> worktree(s): <env>/<repo>, ...", and confirm("Continue?"); both on stdout,
     also under --json, where they precede the first event (output-formats.md, JSON-mode departures); declined or end
     of input -> Aborted, exit 1, nothing read further and nothing written
     # the prompt comes before every refusal check, as in the reference
  6. preflight (below); any refusal -> the refused repos are reported; nothing is written; exit 1
  7. --dry-run -> every target reported reset with its expanded ref; no git write; exit 0
  8. execution (below); exit 0 when every target is reset, else 1 (Reported)
no network: no fetch runs
```

### Preflight

```text
# all-or-nothing (decision:parallel-workspaces/partial-execution); applies under --dry-run too
ambiguous sha    REF as typed matches ^[0-9a-fA-F]{4,40}$ and targets > 1
                 -> every target refused-ambiguous-sha, even under --force; no other check runs
                 # refs/heads/<name> names an all-hex branch across worktrees
expand           per target, before any git call: resolved := expand(REF) (ref-tokens.md)
missing ref      not has_ref(wt, resolved) -> refused-missing-ref; not checked further
--hard, unless --force, per remaining target in order:
  dirty          dirty(wt) -> refused-dirty
  abandonment    unpushed(wt, "HEAD", safety_ref(wt, Head)) > 0 -> refused-abandonment
                 # measured against the old upstream, not REF; soft and mixed run no such check
a probe's repo error ends the command with that RepoError, exit 1, before any effect
refusals         reported as the missing-ref ones in target order, then the safety ones in target order
```

### Execution

```text
# per target in order; tracking and attachment are never touched: a detached HEAD stays detached
  1. previous := commit(wt, "HEAD")
  2. git reset --soft|--mixed|--hard <resolved>
     # --hard overwrites an untracked file in the way of a tracked one (decision:parallel-workspaces/safe-history/pointer-recovery)
  -> reset, previous reported
a failure -> that target failed, with its error; every later target unattempted and untouched; no rollback
# decision:parallel-workspaces/partial-execution — registered deviation: the reference raises the repo error with no
# report of the worktrees already moved or never reached
```

### Result

```text
--json  NDJSON, each line flushed (shape C)
  {"type": "reset_started", "ref": <REF as typed>, "mode": "soft"|"mixed"|"hard", "dry_run": bool}
  {"type": "repo_reset", "env", "repo", "result", "ref", "previous", "error"}     one per reported target, in order
      previous: sha on a moved target, else null; error: message text on failed, else null     # registered deviation
  {"type": "reset_completed", "success": bool}
  target_missing events precede reset_started (target-patterns.md, Missing targets in results)
  refused:  the started event, one repo_reset per refused target, completed with success false
  executed: one repo_reset per target, each written as it is reached, then completed
  nothing matched: the started and completed events alone, success true
text    nothing matched: one line naming PATTERNS
        a table of WORKTREE (<env>/<repo>), RESULT, and REF ("—" for none), then the previous commit of every
        moved worktree
  refused:  that nothing changed, with one hint: target one worktree for ambiguous-sha alone; fetch when any
            missing-ref; --force otherwise
  complete: the count of worktrees, "would reset" under --dry-run, REF as typed, and the mode
  stopped:  the worktree it stopped on with its error, the moved ones, and the unattempted ones
```
