# Target patterns

The `<env>/<repo>` matcher every worktree command selects through, the two rules that list the environments a pattern
can reach, and the refusals and warnings selection itself writes. The pattern shapes per command are the Combination
rules of [command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md); the
glob is `fnmatchcase` of `crates/core/src/pattern_match.rs`, as built; the environments present are `present` of
[location-listing.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/location-listing.md),
Environments present; the repository test is the one of that leaf, The worktree probe. Other specs link this leaf for
selection and never restate it. Reference locations are paths under `tools/winter-cli/src/winter_cli/` of
`winter-workspace` at `127cdf2`. Message wording is the binary's own (`decision:functional-parity`).

## Domain

```rust
// crates/core/src/targets.rs — product:target-patterns, decision:parallel-workspaces/target-selection
// Pure but for the file reads of `select`; starts no program.

/// One parsed `<env>/<repo>` pattern; a bare `<env>` is held as `<env>/*`.
pub(crate) struct Pattern { pub env: String, pub repo: String }

/// Which environments a command's patterns can reach.
pub(crate) enum EnvRule {
    Present,   // ws fetch, pull, merge, push, status: the environments present, by index
    Named,     // ws connect, disconnect, diff, reset, clean: literal env halves by name, then globbed present ones
}

/// A worktree a command acts on. Built for every loaded project entry of each reached environment, in merged
/// order, that some pattern matches; two patterns matching one worktree give one Selected.
pub(crate) struct Selected {
    pub env: String,           // the environment's name
    pub repo: String,          // the entry's resolved name
    pub path: PathBuf,         // <root>/<env>/<repo>, joined, never resolved
    pub main_branch: String,   // the entry's main_branch as loaded (its own, else the workspace's)
    pub pinned: bool,
    pub nested: bool,
}

/// Why a matched worktree is not acted on.
pub(crate) enum Missing { Absent, NotARepository }

pub(crate) struct Selection {
    pub envs: Vec<String>,                  // the reached environments, in rule order, matched or not
    pub worktrees: Vec<Selected>,           // env order, then merged project order
    pub missing: Vec<(Selected, Missing)>,  // matched worktrees left out; each warned once, in the same order
}

pub(crate) fn parse(pattern: &str) -> Result<Pattern, SelectionError>;
pub(crate) fn matches(pattern: &Pattern, env: &str, repo: &str) -> bool;
pub(crate) fn known_env(files: &dyn ReadFiles, root: &Path, registry: &[(String, i64)], name: &str) -> bool;
pub(crate) fn select(files: &dyn ReadFiles, diagnostics: &mut Diagnostics<'_>, root: &Path,
    config: &WorkspaceConfig, registry: &[(String, i64)], patterns: &[Pattern], rule: EnvRule)
    -> Result<Selection, SelectError>;
pub(crate) enum SelectError { Selection(SelectionError), Internal(InternalError) }

// crates/core/src/termination.rs — SelectionError gains, beside EmptyPattern and NotBareName:
NotEnvRepo { pattern: String }      // more than one "/": "invalid pattern <p!r>: expected <env>/<repo> (one '/' max)"
UnknownEnv { env: String }          // "no environment named <env!r>: none is registered and <root>/<env> is not a
                                    //  directory"; registered deviation, decision:parallel-workspaces/target-selection
```

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
# reference: modules/workspace/workspace_sync_service.py:768-780, workspace_push_service.py:66-70,
#            workspace_merge_service.py:254-261, workspace_snapshot_service.py:126 (Present);
#            modules/workspace/handlers/workspace_handler.py:350-375, pattern_match.py:48-68 (Named)
env halves   the env half of each pattern, duplicates dropped
Present      the environments present (location-listing.md), in index order; a pattern reaches each one whose name
             fnmatchcase matches its env half. A literal env half that passes known_env but is not present
             reaches nothing
Named        literal := the env halves that are no glob, sorted by code point; each is reached by name, whether or
             not it is present
             then the environments present whose name matches any glob env half (fnmatchcase(name, half)) and is
             not a literal, sorted by code point
             present is listed only when some env half is a glob
both         the patterns themselves are matched against (env, repo) per worktree: a worktree of a reached env is
             selected when any pattern matches it
```

### Refusals and warnings

```text
# product:target-patterns; decision:parallel-workspaces/target-selection — registered deviations below
env name      after the load and the drift notice, before any git call or write, each literal env half, in
              pattern order, passes env_name::check of ../../environment-lifecycle/spec/env-names.md, else
              CommandError::EnvName, exit 1; so `projects/x` never reaches the source checkouts and `/x` is refused
              (decision:parallel-workspaces/environment-lifecycle/env-name-grammar)
unknown env   then each must pass known_env; the first that does not ends the command with
              SelectionError::UnknownEnv (CommandError::Selection, exit 1)
              ws status ends either refusal as its collection error, exit 2. Nothing has been written to stdout
known_env     name is recorded in the env index registry, or <root>/<name> is a directory, symbolic links followed
              # the reference: Present treats the name as matching nothing, Named builds the env by name and fails
              # at its first git call with NoSuchPathError, after earlier worktrees were written
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
              order, each tested as above with the label standalone/<name> and the fix `winter ws init`
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
