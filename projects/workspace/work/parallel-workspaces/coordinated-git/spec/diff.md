# Diff

The `ws diff` command: one unified diff across the selected worktrees, attributed to repository and environment, that
`git apply` takes back. Its options and refusals are the `ws diff` row of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md); its stdout is
the `ws diff` row of Results another program reads in
[output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md), and its shape A
document is owned here. Selection is [target-patterns.md](./target-patterns.md) with `EnvRule::Named`, default `*/*`;
the drift notice is that of
[drift-warning.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/drift-warning.md). Reference
locations are paths under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`; "WH" is
`modules/workspace/handlers/workspace_handler.py`, "RRR" `modules/workspace/internal/read_repo_repository.py`.

## Domain

```rust
// crates/core/src/diff.rs — product:inspect-workspace-git
pub(crate) enum DiffMode { Uncommitted, Staged, Branch }   // default; --staged; --branch:
                                                           // "uncommitted" | "staged" | "branch"
pub(crate) struct RepoDiff { pub name: String, pub text: String, pub ahead: u64,
    pub files_changed: u64, pub insertions: u64, pub deletions: u64 }
pub(crate) struct EnvDiff { pub env: String, pub repos: Vec<RepoDiff> }
pub(crate) fn diff(ports: &Ports<'_>, diagnostics: &mut Diagnostics<'_>, start: &Path, patterns: &[Pattern],
    mode: DiffMode) -> Result<Vec<EnvDiff>, DiffError>;
pub(crate) enum DiffError { Config(Box<ConfigError>), Selection(SelectionError), Repo(Box<RepoError>),
    Internal(Box<InternalError>) }
// termination.rs — GitOperation gains Diff ("diff"), BranchAheadProbe ("branch ahead probe")
```

## Interfaces

```text
# reference: WH:1060-1132, modules/workspace/env_status_service.py:204-221, RRR:358-423
order      root discovery and the load; the drift notice; selection; then each selected worktree, one at a time,
           in selection order; the first failure ends the command as Termination::Repo, exit 1, with nothing on
           stdout; the result is written whole at the end
per worktree, cwd = its path, P := --src-prefix=a/<repo>/ --dst-prefix=b/<repo>/
  Uncommitted   git diff P;           git diff --numstat
  Staged        git diff --staged P;  git diff --staged --numstat
  Branch        git diff origin/<main>...HEAD P;  git diff origin/<main>...HEAD --numstat
                ahead := git rev-list --count origin/<main>..HEAD; a failure when origin/<main> does not resolve
                is 0, else RepoError GitFailure { BranchAheadProbe, repo }
  a non-zero diff -> RepoError GitFailure { Diff, repo }; in Branch mode a missing origin/<main> is such a failure
  numstat       one file per non-empty line; "-" (binary) counts 0 insertions or deletions
  kept          a repository whose diff text is empty is left out; in Branch mode so is a pinned one with ahead 0
  an environment with no repository kept is left out
untracked files are in no mode, as git diff leaves them out
```

```text
--json, shape A
{"patterns": [<as given, or ["*/*"]>], "mode": "<mode>",
 "results": [{"env": <name>, "repos": [{"name", "files_changed", "insertions", "deletions"}]}],
 "missing": [entry...]}       # missing: target-patterns.md, Missing targets in results
text
  nothing kept -> nothing written, exit 0
  more than one environment and not --no-headers: the env name, bold, before each environment's repositories,
  and a blank line between environments
  per repository, unless --no-headers: "=== <repo> ===", or "=== <repo> (+<ahead> commit(s)) ===" in Branch mode
  with ahead > 0; then the diff text with one final newline; a blank line between repositories
  with --no-headers the output is the diffs alone, which `git apply` takes from the environment directory
```
