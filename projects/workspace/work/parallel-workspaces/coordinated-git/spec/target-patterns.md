# Target patterns

The `<env>/<repo>` matcher every worktree command selects through, and the refusals and warnings selection itself
writes. The pattern shapes per command are the Combination rules of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md); the glob is
`fnmatchcase` of `crates/core/src/pattern_match.rs`, as built; the two rules that list the environments a pattern can
reach, `EnvRule`, `known_env`, and the unknown-environment refusal are `reach` of
[env-selection.md](../../environment-lifecycle/spec/env-selection.md),
`epic:parallel-workspaces/environment-lifecycle`'s; the repository test is the one of
[location-listing.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/location-listing.md), The
worktree probe. Other specs link this leaf for selection and never restate it. Reference locations are paths under
`tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`. Message wording is the binary's own
(`decision:functional-parity`).

## Domain

```rust
// crates/core/src/targets.rs — product:target-patterns, decision:parallel-workspaces/target-selection
// Pure but for the file reads of `select`; starts no program.

/// One parsed `<env>/<repo>` pattern; a bare `<env>` is held as `<env>/*`.
pub struct Pattern { pub env: String, pub repo: String }

// EnvRule, known_env, and SelectionError::UnknownEnv are env-selection.md's; the Rule column below names each
// command's EnvRule.

/// A worktree a command acts on. Built for every loaded project entry of each reached environment, in merged
/// order, that some pattern matches; two patterns matching one worktree give one Selected.
pub struct Selected {
    pub env: String,           // the environment's name
    pub repo: String,          // the entry's resolved name
    pub path: PathBuf,         // <root>/<env>/<repo>, joined, never resolved
    pub main_branch: String,   // the entry's main_branch as loaded (its own, else the workspace's)
    pub pinned: bool,
    pub nested: bool,
}

/// Why a matched worktree is not acted on.
pub enum Missing { Absent, NotARepository }

pub struct Selection {
    pub envs: Vec<String>,                  // the reached environments, in rule order, matched or not
    pub worktrees: Vec<Selected>,           // env order, then merged project order
    pub missing: Vec<(Selected, Missing)>,  // matched worktrees left out; each warned once, in the same order
}

pub fn parse(pattern: &str) -> Result<Pattern, SelectionError>;
pub fn matches(pattern: &Pattern, env: &str, repo: &str) -> bool;
pub fn select(files: &dyn ReadFiles, diagnostics: &mut Diagnostics<'_>, root: &Path,
    config: &WorkspaceConfig, registry: &[(String, i64)], patterns: &[Pattern], rule: EnvRule)
    -> Result<Selection, SelectError>;
pub enum SelectError { EnvName(EnvNameError), Selection(SelectionError), Internal(InternalError) }
// an entry's own error carries each variant as its EnvName, Selection, and Internal (Refusals and warnings)

/// The standalones --standalone and --all reach: every declared standalone, in merged order, each tested once as
/// Refusals and warnings, standalones, gives; a missing one is warned there and is in `missing`, never in `present`.
pub struct Standalones { pub present: Vec<(String, PathBuf)>, pub missing: Vec<(String, Missing)> }   // name, path
pub fn standalones(files: &dyn ReadFiles, diagnostics: &mut Diagnostics<'_>, root: &Path, config: &WorkspaceConfig)
    -> Standalones;

/// Missing targets in results: one value per missing worktree, then one per missing standalone, in that order, the
/// order select and standalones warned them. The command file places them: the `missing` member of a shape A
/// document, or the target_missing events of a shape C run.
pub fn missing_entries(worktrees: &[(Selected, Missing)], standalones: &[(String, Missing)]) -> Vec<serde_json::Value>;
pub fn target_missing_events(worktrees: &[(Selected, Missing)], standalones: &[(String, Missing)])
    -> Vec<serde_json::Value>;

// crates/core/src/termination.rs — SelectionError gains, beside EmptyPattern and NotBareName:
NotEnvRepo { pattern: String }      // more than one "/": "invalid pattern <p!r>: expected <env>/<repo> (one '/' max)"
```

Visibility: `Pattern`, `Selected`, `Missing`, `Selection`, `SelectError`, `Standalones`, `parse`, `matches`, `select`,
`standalones`, `missing_entries`, and `target_missing_events` are `pub` inside their private module and re-exported by
name from `lib.rs`, the module's names on a `pub use` line of their own, with every type their signatures name
(`context/architecture/changes.md`, A domain module, of `winter-workspace`), so the slice that lands them tests them
before a command calls them (`context/standards/testing.md`) and the gate finds nothing unused without an `allow`
(`context/standards/rust.md`). `epic:parallel-workspaces/safe-history`'s probes, which are `pub` too, name `Selected`.

## Interfaces

### Grammar and match

```text
# reference: modules/workspace/command.py:99-103, modules/workspace/pattern_match.py:9-11,86-101
parse(p)
  p empty                      -> SelectionError::EmptyPattern
  p holds more than one "/"    -> SelectionError::NotEnvRepo
  p holds no "/"               -> Pattern { env: p, repo: "*" }
  otherwise                    -> split at the "/": Pattern { env: before, repo: after }; either half may be empty
matches(pat, env, repo)        fnmatchcase(env, pat.env) and fnmatchcase(repo, pat.repo)
  a half is a glob when it holds "*", "?", or "[" (pattern_match::is_glob), else an exact, case-sensitive name;
  "*" never crosses into the other half, and "**" is "*": fnmatchcase folds a run of "*" into one
parse runs where Combination rules places the pattern-shape check: before root discovery, the load, and any git call
```

### Reaching environments

```text
env halves   the env half of each pattern, duplicates dropped
reached      env_selection::reach(files, root, config, registry, env halves, the command's EnvRule)
             (env-selection.md); its ReachError::Internal is SelectError::Internal
both         the patterns themselves are matched against (env, repo) per worktree: a worktree of a reached env is
             selected when any pattern matches it
```

### Refusals and warnings

```text
# product:target-patterns; decision:parallel-workspaces/target-selection — registered deviations below
env name      after the load and the drift notice, before any git call or write, each literal env half, in
              pattern order, passes env_name::check of ../../environment-lifecycle/spec/env-names.md, else
              SelectError::EnvName, which the entry's error carries as its EnvName variant and the command file maps
              to CommandError::EnvName, exit 1; so `projects/x` never reaches the source checkouts and `/x` is refused
              (decision:parallel-workspaces/environment-lifecycle/env-name-grammar)
unknown env   then reach's refusal (env-selection.md): the first literal env half that names no environment ends
              the command with SelectionError::UnknownEnv (CommandError::Selection, exit 1)
              ws status ends either refusal as its collection error, exit 2. Nothing has been written to stdout
missing       each selected worktree, in selection order, is tested once:
                <path> does not exist                    -> Missing::Absent
                <path> is no repository (location-listing.md's test, no parent search)
                                                         -> Missing::NotARepository
              a missing worktree is moved to Selection.missing and warned, through Diagnostics::notice:
                "warning: <env>/<repo> — missing on disk (run `winter ws init <env>`)"
                "warning: <env>/<repo> — not a git repository"
              no command starts git in a missing worktree, so git never searches a directory above it
              # registered deviation, decision:parallel-workspaces/target-selection: the reference warns for
              # fetch, pull, and merge only; push and status read
              # an absent worktree as clean zeros, and connect, disconnect, and diff fail on it
no target     selection never infers a target from the working directory; with no PATTERNS a command takes the
              default its own leaf states, or refuses as Combination rules says
standalones   patterns never select a standalone: --standalone and --all reach every declared standalone, in merged
              order, through `standalones`, each tested once as above with the label standalone/<name> and the fix
              `winter ws init`:
                "warning: standalone/<name> — missing on disk (run `winter ws init`)"
                "warning: standalone/<name> — not a git repository"
              a missing standalone is in Standalones.missing, named in the result as Missing targets in results
              gives, and no command starts git in it; ws status reads its standalones through `standalones` too
```

| Command         | No PATTERNS                   | Rule    | Leaf                                    |
| --------------- | ----------------------------- | ------- | --------------------------------------- |
| `ws fetch`      | `*/*`                         | Present | [refresh.md](./refresh.md)              |
| `ws pull`       | `*/*`                         | Present | [refresh.md](./refresh.md)              |
| `ws merge`      | refused unless `--standalone` | Present | [merge.md](./merge.md)                  |
| `ws push`       | `*/*`                         | Present | [push.md](./push.md)                    |
| `ws status`     | every worktree                | Present | [status.md](./status.md)                |
| `ws diff`       | `*/*`                         | Named   | [diff.md](./diff.md)                    |
| `ws connect`    | usage error (parser)          | Named   | [tracking.md](./tracking.md)            |
| `ws disconnect` | usage error (parser)          | Named   | [tracking.md](./tracking.md)            |
| `ws reset`      | usage error (parser)          | Named   | `epic:parallel-workspaces/safe-history` |
| `ws clean`      | usage error (parser)          | Named   | `epic:parallel-workspaces/safe-history` |

A command that matches nothing writes the result its own leaf gives for no target, and exits as that leaf says; the
selection writes nothing for it.

### Missing targets in results

```text
# product:explain-results, product:target-patterns; registered deviation, decision:parallel-workspaces/target-selection:
# the reference names a skipped target in a warning at most, never in its --json output
entry       {"env": <env, or "standalone">, "repo": <name>, "reason": "absent" | "not_a_repository"}, one per entry of
            Selection.missing, and per missing standalone, in the order they were warned: missing_entries
shape A     the command's document gains the member "missing": [entry...], [] when none
shape C     one {"type": "target_missing", "env", "repo", "reason"} event per entry, written before the run's started
            event; a run that writes no other event under --json, as for nothing to do, still writes these:
            target_missing_events
applies to  every command of the table above, and ws checkout of epic:parallel-workspaces/safe-history; its leaf
            places the member or the events, and output-formats.md's vocabulary rows gain target_missing first
ws status   excepted: the published v1 schema admits no extra member, so a missing worktree there is the warning
            above and its exit code (status.md, Exit code), as target-selection decides
the warnings above are written in every mode, also when the result names the entry
output-formats.md = ../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md
```
