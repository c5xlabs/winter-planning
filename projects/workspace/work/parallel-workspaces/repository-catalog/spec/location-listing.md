# Location listing

The `ws worktrees` command: every checkout location that exists — feature worktrees, standalones, and the workspace
itself — for a caller such as an editor integration to jump to, and with `--status` the Git counts of each. Its
arguments are the `ws worktrees` row of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), Commands.
Reference locations are paths under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `b57b020`; "WH" is
`modules/workspace/handlers/workspace_handler.py`.

## Interfaces

```text
# product:inventory-presence — reference: modules/workspace/command.py:246-274, WH:836-958,1527-1563
ws worktrees [--json] [--status]
order      1. workspace root, then the load       configuration-model.md, Root discovery, then Load
           2. the environments present            Environments present below
           3. per environment, in order: its worktree locations; with --status, their probes
           4. the standalone locations
           5. the workspace location; with --status, its probe
           6. the result on stdout, written whole at the end: a failure in 2 to 5 writes nothing to stdout
no drift warning; no refusal of its own; nothing is created, written, locked, or recorded; no fetch
configuration-model.md = ../../../../delivered/parallel-workspaces/workspace-configuration/spec/configuration-model.md
```

```text
locations, in this order; a location is listed only when it exists and is a directory, symbolic links followed
  worktree     for each environment present E, in order, for each loaded project entry P, in merged order
               (pinned ones included; two entries with one resolved name give two locations):
                 path <root>/<E>/<P's resolved name>, label "<E>/<P's resolved name>"
  standalone   for each loaded standalone entry S, in merged order, extension = false included:
                 path <root>/<S's path when not empty, else its resolved name>, label its resolved name
  workspace    always last, and always present: path <root>, label "<workspace>"
paths are normalised as repository-list.md, locations, says; nothing is resolved
the implicit product and harness repositories are not listed
```

### Environments present

```text
# reference: modules/workspace/internal/read_workspace_repository.py:51-139
# decision:parallel-workspaces/environment-lifecycle/discovery-authority owns any change to this rule
candidates   the loaded env_aliases in their order, then the names the env index registry records
             (../../../../delivered/parallel-workspaces/workspace-configuration/spec/lock-and-state.md, the state
             file; a missing or unparsable file records none) in the state file's order; a name already a candidate
             is not repeated
present      <root>/<name> is a directory, symbolic links followed, and one of its entries is a directory, symbolic
             links followed, named as the resolved name of a loaded project entry
order        by index, ascending: the index ../../environment-lifecycle/spec/index-lookup.md's `lookup` answers for
             the name; candidates of equal index keep their candidate order
not present  a directory that is neither an alias nor recorded; a candidate that holds no project directory
listing      a candidate directory that exists and cannot be listed ends the command as InternalError::Unexpected,
             exit 1, as the reference's unhandled OSError
```

### With `--status`

```text
# reference: modules/workspace/env_status_service.py:89-123,191-199; WH:848-853,912-930;
#            modules/workspace/internal/read_repo_repository.py:271-351
programs     every git call goes through the program port: child-processes.md
worktree     for each environment present, in order, the worktree probe below runs for every loaded project entry,
             whether or not its location exists, with name = P's resolved name and cwd = the location
  failure    when a probe of an environment fails, every probe of that environment still runs, the first failure in
             project order ends the command as Termination::Repo, exit 1, and no environment after it is probed
             (env_status_service.py:104-116). Which probes of one environment run in parallel is free.
  counts     a worktree location takes the counts of the probe of its entry's resolved name; two entries with one
             resolved name both show the counts of the later entry's probe, as the reference keys them by name
standalone   ahead, behind, dirty are null; no git call
workspace    cwd = <root>; ahead, behind, dirty are all null when <root> is no repository (below), when the branch
             below is null, and when any git call below exits not 0; otherwise they are the values below, and the
             command goes on either way
  branch             HEAD is a symbolic ref -> its branch name; detached -> null
  dirty              the distinct paths with staged changes against HEAD (none when HEAD has no commit yet), with
                     those with unstaged changes, plus the number of untracked files, each file of an untracked
                     directory counted
  ahead, behind      the upstream, read as GitPython's tracking_branch() does (git/refs/head.py:190-206): with
                     branch.<branch>.remote = R and branch.<branch>.merge = M both set, the operand is the text
                     "R/" followed by M without a leading "refs/heads/"; else there is none -> 0, 0.
                     ahead = git rev-list --count <operand>..HEAD; behind = git rev-list --count HEAD..<operand>.
                     A local upstream, R = ".", gives the operand "./<branch>", whose rev-list fails -> null
```

### The worktree probe

Shared: `ws status` of `epic:parallel-workspaces/coordinated-git` runs the same probe for its worktrees, as the
reference does (`read_repo_repository.py:232-252`), and adds pieces of its own; its spec links this section rather than
copying it.

```text
# reference: modules/workspace/internal/read_repo_repository.py:56-154,505-620,668-673
input        a location L, a repository name, and its main branch: the entry's main_branch when not empty, else the
             workspace main_branch
  not a repository   L does not exist, or is no repository (below) -> ahead 0, behind 0, dirty 0, and no git call
  1. git status --porcelain=v2 --branch --untracked-files=all -z
       exit not 0 -> RepoError "status probe failed for <name>"
       dirty = the number of distinct paths of the records of type 1 or 2 whose X or Y is not "." and of type u
               (for a type 2 record, its new path), plus the number of "?" records; "!" records and headers count
               nothing
  2. git rev-list --left-right --count origin/<main>...HEAD   ->  "<behind>\t<ahead>"
       exit not 0 -> git rev-parse --verify --quiet origin/<main>
                       exit 0     -> RepoError "main-branch ahead/behind probe failed for <name>"
                       otherwise  -> ahead 0, behind 0
  the upstream of the branch plays no part; a detached HEAD is probed as any other
```

```text
# a repository, as GitPython 3.1.46's Repo(path) decides with no parent search — git/repo/base.py:238-288,
# git/repo/fun.py:57-121
a location L is a repository when any of these holds, symbolic links followed:
  L is a Git directory: it holds the directories objects and refs, and HEAD as a file or as a link whose target
      starts with "refs"
  L/.git is a Git directory, as above
  L/.git is a file starting "gitdir: " whose path, relative to L, is such a directory or such a file, recursively
  L/.git is a regular file whose every line reads "<key>: <value>" and one key is gitdir, whatever it names
otherwise it is none: no .git entry, an empty .git directory, a .git file of another form
a repository whose git status fails — a bare repository at L, a gitdir naming nothing — is a RepoError, not zeros
```

```text
# tracing — reference: modules/workspace/internal/git_operation.py:14-86, read_repo_repository.py:271-272,505-506,
#           core/internal/otel_command_tracer.py:109-114,219-234,281-320; engineering:observability-isolation
with tracing on and --status, one operation span per probe, a child of the command span:
  worktree     one per probe: every project entry of each environment probed, whether or not its location exists
               name "git status"; attributes winter.repo = P's resolved name, winter.env = E
  workspace    one, when the workspace probe runs
               name "git status"; attribute winter.repo = the root directory's base name
  a span covers every git call of its probe, and a probe that makes none still has its span
  failed       a probe that ends in a RepoError: status ERROR with no description, error.type "RepoError"; for the
               workspace probe as well, though the listing goes on with nulls
  never        a span per git process, a span without --status, an argument, a path, or output as an attribute
each span is ended before the command span and is sent in the command span's one export (tracing.md, The binary's
part); recording follows the command span's sampling. tracing.md =
../../../../delivered/parallel-workspaces/component-runtime/spec/tracing.md
```

```text
--json     one Shape A document, an array (../../../../delivered/parallel-workspaces/component-runtime/spec/
           output-formats.md, Shape per command), one object per location in order, keys:
             kind    "worktree" | "standalone" | "workspace"
             env     E for a worktree; null otherwise
             repo    P's resolved name for a worktree; null otherwise
             name    null for a worktree; the resolved name for a standalone; the root directory's base name for
                     the workspace
             label   as above
             path    the location, absolute
           with --status, three keys more: "ahead", "behind", "dirty", each an integer or null as above
           without --status the three keys are absent, not null (output-formats.md, JSON-mode departures)
human      one row per location, in order, carrying the label, the kind, the path, and with --status the counts;
           layout is the binary's own (output-formats.md, Human output)
exit       0
```

Reference human form, an example under `decision:functional-parity`: a borderless table whose header row names `LABEL`,
`KIND`, and `PATH`, and `STATUS` with `--status`, with columns two spaces apart. A status cell is `+<ahead> -<behind>`
and `[+<dirty>]` when dirty is above zero, a null part left out, and `=` when nothing is left (WH:1549-1563): a clean
worktree shows `+0 -0`, a standalone `=`.

```text
LABEL        KIND        PATH                                STATUS
alpha/api    worktree    /home/u/ws/alpha/api                +2 -1 [+3]
ctx          standalone  /home/u/ws/.winter/ext/context      =
<workspace>  workspace   /home/u/ws                          +0 -0
```

Failures: a missing root and every configuration fault end as the `ConfigError` of the section that defines it; a failed
worktree probe as `Termination::Repo` holding the `RepoError` of
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
Domain, with the program, subcommand, arguments, cwd, exit, and stderr of the failed call, written once
(`decision:parallel-workspaces/component-runtime/git-failure-rendering`); a git that cannot be started as
[child-processes.md](./child-processes.md) says.

## Domain

```rust
// crates/core/src/locations.rs
pub(crate) enum LocationKind { Worktree, Standalone, Workspace }
pub(crate) struct Location {
    pub kind: LocationKind,
    pub env: Option<String>, pub repo: Option<String>, pub name: Option<String>,
    pub label: String, pub path: PathBuf,
    pub status: Option<Counts>,                    // Some exactly when --status was given
}
pub(crate) struct Counts { pub ahead: Option<u64>, pub behind: Option<u64>, pub dirty: Option<u64> }
pub(crate) fn list(files: &dyn ReadFiles, programs: &dyn RunPrograms, tracer: &dyn CommandTracer, start: &Path,
    with_status: bool)
    -> Result<Vec<Location>, ListError>;
pub(crate) enum ListError { Config(ConfigError), Repo(RepoError), Internal(InternalError) }

// crates/core/src/environments.rs — the environments present; reused by the commands that list environments
pub(crate) fn present(files: &dyn ReadFiles, root: &Path, config: &Config, registry: &[(String, i64)])
    -> Result<Vec<String>, InternalError>;
```

`registry` is the env index registry in the state file's order; reading it keeps that order.

## Open questions

- The workspace probe is GitPython's `index.diff("HEAD")`, `index.diff(None)`, `untracked_files`, and
  `tracking_branch()` (3.1.46), not a git command line of the reference's own; how a staged rename and a path with both
  staged and unstaged changes count is read from GitPython, not run. The slice that builds the probe records a scenario
  with each, and the dirty rule above follows the record.
- If `decision:parallel-workspaces/environment-lifecycle/discovery-authority` is made with a rule other than the one
  under Environments present, environment-lifecycle engineering moves that rule into its own spec and this leaf links
  it.
