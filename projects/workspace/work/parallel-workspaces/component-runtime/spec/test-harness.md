# Test harness

The `winter-workspace-test-support` package: the isolated tree every process-level test runs in, the pinned environment,
the bounded process runner, the git fixture helpers, and the workspace builder. The scenario format and its record and
replay machinery live in the same package and are specified by
[scenario-format.md](../../../rust-adoption/parity-validation/spec/scenario-format.md) and
[record-and-replay.md](../../../rust-adoption/parity-validation/spec/record-and-replay.md). The package's place in the
workspace is [repository-layout.md](./repository-layout.md).

The package depends on no other workspace package. It is written fresh in this repository from the interfaces below; no
source of the platform repository's harness is copied or adapted, under
`decision:parallel-workspaces/component-runtime/harness-provenance`.

## System

```text
# engineering:testable-boundaries, engineering:rust-adoption/parity-validation/disposable-runtime-proof
crates/test-support/src/
  lib.rs          facade: named re-exports only
  tree.rs         TempTree
  env.rs          PinnedEnv
  process.rs      Invocation, invoke, Observation, InvocationFailure
  git.rs          Git, CommitSpec, GitError
  workspace.rs    WorkspaceBuilder, Workspace
  scenario.rs, scenario/…     scenario-format.md, record-and-replay.md
```

### Tiers that use it

| Tier       | Lives in                         | Launches a process | Uses                                                                          |
| ---------- | -------------------------------- | ------------------ | ----------------------------------------------------------------------------- |
| Unit       | `#[cfg(test)]` beside the code   | no                 | nothing from this package                                                     |
| Component  | `crates/core/tests/<subject>.rs` | git only           | `TempTree`, `Git`, `WorkspaceBuilder`; calls `run` with in-memory handles     |
| Executable | `crates/cli/tests/<subject>.rs`  | the built binary   | all of the above plus `invoke`, with `env!("CARGO_BIN_EXE_winter-workspace")` |
| Scenario   | `crates/cli/tests/scenarios.rs`  | the built binary   | the scenario runner                                                           |

## Domain

### The isolated tree

```text
<parent>/winter-workspace-test-<random>/        the ROOT; canonical path (symlinks resolved)
  ws/              the workspace directory; its basename is observable, so it is never the random directory
  remotes/         bare repositories, one <name>.git each
  home/            HOME            home/.config/   XDG_CONFIG_HOME
  cache/           XDG_CACHE_HOME
  tmp/             TMPDIR
  bin/             first on PATH; stub programs
  calls/           what stub programs record
  git-template/    empty; GIT_TEMPLATE_DIR
  gitconfig        GIT_CONFIG_GLOBAL

parent        WINTER_TEST_TMPDIR when set, else the system temporary directory; canonicalised before any check
refused       creation fails, creating nothing, when the parent, or any ancestor of it, holds
                .winter/config.toml
                    "no isolated tree is created in <parent>: <path> is a workspace configuration"
                .git, as a directory or a file; or HEAD beside the directories objects and refs, which is a git
                directory, a bare repository included
                    "no isolated tree is created in <parent>: <path> is a git repository"
              <path> is the entry found: the configuration file, the .git entry, or the git directory. The
              workspace check runs first, so a workspace that is also a repository is reported as a workspace.
              No walk upward from inside a tree can reach a real workspace or a real repository, under
              decision:parallel-workspaces/component-runtime/git-discovery-containment
path charset  the canonical ROOT matches [A-Za-z0-9/._-]+ ; anything else fails creation, so the ROOT can be
              replaced textually inside JSON without unescaping
umask         the process runner starts every child with umask 022
removal       on drop, recursively; kept, with its path printed to stderr, when WINTER_TEST_KEEP=1
```

### The pinned environment

Every child the harness starts, git and the binary alike, gets exactly this environment and nothing inherited.

| Variable                                    | Value                                                   |
| ------------------------------------------- | ------------------------------------------------------- |
| `PATH`                                      | `<ROOT>/bin:<directory of git>:/usr/bin:/bin`           |
| `HOME`                                      | `<ROOT>/home`                                           |
| `XDG_CONFIG_HOME`                           | `<ROOT>/home/.config`                                   |
| `XDG_CACHE_HOME`                            | `<ROOT>/cache`                                          |
| `TMPDIR`                                    | `<ROOT>/tmp`                                            |
| `GIT_CONFIG_GLOBAL`                         | `<ROOT>/gitconfig`                                      |
| `GIT_CONFIG_NOSYSTEM`                       | `1`                                                     |
| `GIT_TEMPLATE_DIR`                          | `<ROOT>/git-template`                                   |
| `GIT_CEILING_DIRECTORIES`                   | `<ROOT>/..`                                             |
| `GIT_TERMINAL_PROMPT`                       | `0`                                                     |
| `GIT_AUTHOR_NAME`, `GIT_AUTHOR_EMAIL`       | `Fixture Author`, `author@fixture.invalid`              |
| `GIT_COMMITTER_NAME`, `GIT_COMMITTER_EMAIL` | `Fixture Committer`, `committer@fixture.invalid`        |
| `GIT_AUTHOR_DATE`, `GIT_COMMITTER_DATE`     | `2001-01-01T00:00:00+0000`; fixture commits override it |
| `TZ`                                        | `UTC`                                                   |
| `LC_ALL`, `LANG`                            | `C.UTF-8`                                               |
| `NO_COLOR`, `TERM`, `COLUMNS`, `LINES`      | `1`, `dumb`, `80`, `24`                                 |
| `OTEL_SDK_DISABLED`                         | `true`                                                  |

```text
<directory of git>   the directory of the first `git` on the test process's own PATH, resolved once
ceiling              git examines neither a directory GIT_CEILING_DIRECTORIES names nor any above it, and an entry
                     binds only a working directory strictly below it, so the entry is the parent of <ROOT> and not
                     <ROOT>: every working directory at or below <ROOT> is bounded, a repository a fixture makes
                     at <ROOT>/ws or deeper is still found from inside it, and <ROOT> itself never holds .git.
                     The parent is written <ROOT>/.., which git resolves to its real path before comparing, so the
                     digest holds no host path.
never passed         any WINTER_*, TRACEPARENT, TRACESTATE, any other OTEL_*, GIT_SSH_COMMAND, GIT_DIR, GIT_WORK_TREE,
                     VIRTUAL_ENV, PYTHONPATH, PYTHONHOME, FORCE_COLOR — unless a test adds one explicitly
gitconfig            [url "<ROOT>/remotes/"] insteadOf = https://fixture.invalid/        and nothing else
digest               PinnedEnv::digest(): SHA-1 over the sorted NAME=VALUE lines with <ROOT> and <directory of git>
                     left as those placeholders, and over the gitconfig text likewise
```

```text
names the package never spells
  "never passed" is a consequence, not a list in code: the child's environment is cleared and then holds the table
  above, so nothing has to be removed by name and no source file holds a deny-list.
  No file under crates/test-support, crates/core, or crates/cli — source, test, comment, or data — contains
  WINTER_REFERENCE_BIN.
  No file under crates/test-support, nor crates/cli/tests/scenarios.rs, contains the word "python" in any letter
  case, the two interpreter variables above included. The word rule stops there: it keeps the code that runs
  scenarios unable to name the reference tool's language, and it does not bind crates/core or crates/cli/src,
  where the product names that language wherever a contract of its own does.
  A test that proves nothing leaks sets a variable of its own, WINTER_TEST_LEAK_PROBE, on itself and asserts the
  child does not see it.
  Both rules are checked mechanically once crates/dev/tests/architecture.rs gains the substring rules of
  record-and-replay.md, System; every change before that keeps to them, so that check starts on a tree that passes.
```

```text
fixtures that fail by design
  A test of this package that must fail, or that leaves a tree its own drop cannot remove, is a fixture: it runs
  only as a child test process of the test that asserts on it. That parent starts it through invoke under its own
  tree's pinned environment, with WINTER_TEST_TMPDIR naming a directory inside that tree and
  WINTER_TEST_FIXTURE=<the fixture's test name> added.
  A fixture that does not find WINTER_TEST_FIXTURE equal to its own name returns at once and passes, having
  created nothing. A run that selects ignored tests starts every fixture in the outer test process; there its tree
  would land under the host's temporary directory, with no parent test to make it removable again.
```

## Interfaces

```rust
// tree.rs
pub struct TempTree { /* … */ }
impl TempTree {
    pub fn new() -> std::io::Result<Self>;
    pub fn root(&self) -> &Path;
    pub fn workspace_dir(&self) -> PathBuf;                    // <ROOT>/ws
    pub fn env(&self) -> PinnedEnv;
    pub fn invocation(&self, executable: &Path) -> Invocation; // pinned env, cwd <ROOT>/ws, 30 s deadline
}

// env.rs
pub struct PinnedEnv { /* BTreeMap<OsString, OsString> */ }
impl PinnedEnv {
    pub fn set(&mut self, key: &str, value: &str);
    pub fn digest(&self) -> String;
}

// process.rs
pub struct Invocation {
    pub executable: PathBuf,              // absolute; a relative path is InvalidInvocation
    pub args: Vec<OsString>,
    pub cwd: PathBuf,
    pub env: PinnedEnv,
    pub stdin: Vec<u8>,                   // written, then closed; empty means end of input at once
    pub timeout: Duration,
}
pub struct Observation { pub stdout: Vec<u8>, pub stderr: Vec<u8>, pub exit: Exit }
pub enum Exit { Code(i32), Signal(i32) }  // a non-zero exit is an observation, not a failure
// The failure has these three fields and no other, private ones included: any module of the package and any test
// builds one as a literal and matches on all of it.
pub struct InvocationFailure { pub kind: FailureKind, pub stdout: Vec<u8>, pub stderr: Vec<u8> }
pub enum FailureKind { InvalidInvocation, Spawn(String), Timeout, OutputLimit, Cleanup }
pub fn invoke(invocation: Invocation) -> Result<Observation, InvocationFailure>;
```

```text
invoke
  environment   cleared, then exactly Invocation.env
  streams       stdout and stderr are drained concurrently into separate buffers; 8 MiB per stream, then OutputLimit
  group         the child leads its own process group
  spawn         an executable that cannot be started is FailureKind::Spawn carrying the cause,
                "<executable> in <cwd>: <error of the operating system>"; the failure's text is
                "the executable could not be started: " followed by the cause
  deadline      on expiry: SIGKILL to the group, reap, FailureKind::Timeout with the output captured so far
  unwind        a panic or early return in the caller kills and reaps the group
  never         a shell; an executable looked up on PATH; a relative executable
```

```rust
// git.rs — every call runs `git` under the tree's pinned environment with a 30 s deadline
pub struct Git<'t> { /* &'t TempTree */ }
pub struct CommitSpec {
    pub message: String,
    pub files: Vec<(String, Vec<u8>)>,    // path relative to the repository, content; replaces the file
    pub delete: Vec<String>,
    pub branch: Option<String>,           // commit on this branch, creating it from HEAD when absent
    pub tag: Option<String>,              // lightweight tag on the new commit
}
impl<'t> Git<'t> {
    pub fn new(tree: &'t TempTree) -> Result<Self, GitError>;                       // fails when no git is on PATH
    pub fn run(&self, cwd: &Path, args: &[&str]) -> Result<String, GitError>;      // stdout; non-zero exit is GitError
    pub fn init(&self, path: &Path, branch: &str) -> Result<(), GitError>;         // the init command below
    pub fn commit(&self, repo: &Path, spec: &CommitSpec) -> Result<String, GitError>;   // returns the object id
    pub fn bare_remote(&self, name: &str, default_branch: &str, commits: &[CommitSpec]) -> Result<PathBuf, GitError>;
    pub fn clone_remote(&self, name: &str, dest: &Path) -> Result<(), GitError>;   // from https://fixture.invalid/<name>.git
    pub fn add_worktree(&self, repo: &Path, dest: &Path, branch: &str, start: &str) -> Result<(), GitError>;
    pub fn push_to_remote(&self, name: &str, spec: &CommitSpec) -> Result<String, GitError>;  // a commit made elsewhere
}
```

```text
confinement     # decision:parallel-workspaces/component-runtime/fixture-helper-guarantees
                Every path a helper takes — a repository, a working directory, a destination, a file a CommitSpec
                writes or deletes — is checked before the helper has any effect: until every path of the call has
                passed, no file is written or removed, no directory is made, and git is not started. A path that
                fails is GitError::Unconfined naming it. The workspace builder's writes are held to the same rule.
                  base path     a repository, a working directory, or a destination. It is absolute, and a
                                relative one is refused: it would be read against the test process's own working
                                directory. No component is "..". The deepest ancestor of it that exists, the path
                                itself included, canonicalised (symbolic links resolved), lies at or under the
                                canonical ROOT. An entry exists when it is there as a link, whatever the link
                                points to; one that cannot be canonicalised is refused.
                  file path     relative to its repository, not empty, every component a plain name; joined to
                                the repository, it passes the base-path check
                  destination   of init, clone_remote, and add_worktree: a base path that is strictly below the
                                ROOT, as given and as resolved. The ROOT is never a repository: the ceiling of
                                the pinned environment rests on <ROOT> never holding .git.
                  remote name   one path segment
                The check is one crate-private function in tree.rs, beside the root it checks against. git.rs,
                workspace.rs, and the scenario fixture builder call it and hold no second one, so a component
                test that uses the helpers directly has the same boundary as a recorded scenario.
init command    git init --object-format=sha1 --ref-format=files -b <branch> <path>
                for every repository the helpers create: init, the bare repository, each scratch repository, the
                workspace root. Both formats are named so that a git whose built-in defaults differ makes the same
                object ids and the same config: `git config --local --list` of a fixture repository holds no
                `extensions.` key. A git that lacks either option fails the init, and no fixture is built with it.
clone command   git clone --ref-format=files <url> <dest>
                A clone takes its object format from the remote, so only the ref storage is named. A worktree
                shares the storage of its repository and names neither.
commit dates    the n-th commit the helpers make in one tree, n from 0, carries author and committer date
                2000-01-01T00:00:00+0000 plus n minutes, so object ids are equal across runs and across ROOTs
                and no two fixture commits share a timestamp
                n counts commits that exist, not attempts: a commit git refuses consumes no date, and the next
                commit made carries it. The count is state of the tree and not of the process: TempTree holds it,
                every Git over one tree continues one count, two trees never share one, and it ends with the tree.
                Taking the date, making the commit, and advancing the count are one step, so two commits made in
                one tree at once still never share a date.
pinned ids      the fixture "pinned": in a fresh tree, before any other commit, bare_remote("pinned", "main", …)
                  1. message "one"     files a.txt = "one\n"
                  2. message "two"     files b.txt = "two\n"
                  3. message "three"   delete a.txt, tag v1
                tests/git.rs holds the three commit ids of main, oldest first, as literal constants and asserts
                them. The ids follow from this block, the commit dates, the init command, and the identities of
                the pinned environment, and from nothing of the host, so they are equal everywhere. The constants
                are the tripwire for host influence: a host configuration, a changed default format, an identity
                or a date that leaked. Two builds on one host cannot show that, because a leak reaches both. The
                constants change only together with a rule of this spec that determines them.
remote URL      every clone's remote.origin.url is https://fixture.invalid/<name>.git; the gitconfig rule maps it
                to <ROOT>/remotes/<name>.git at transport time, so no ROOT is ever committed
                A clone of a name the tree has no remote for fails on the mapped path: git's stderr names
                <ROOT>/remotes/<name>.git and never fixture.invalid, and no host name was resolved.
where the ROOT  outside object storage, the files of a fixture repository that may hold the ROOT are a worktree's
may appear      gitdir pointers, and FETCH_HEAD once a fetch has run, where git may write the URL as rewritten for
                transport. No other file of a repository the helpers build holds it.
bare_remote     1. the init command with --bare, at <ROOT>/remotes/<name>.git, on <default_branch>
                2. build the commits in a scratch repository under <ROOT>/tmp
                3. from the scratch repository, push refs/heads/* and refs/tags/* to the bare repository, naming
                   it by its path on the command line; no remote is added to either repository
                4. remove the scratch repository
                The bare repository is never the product of a clone, so it has no remote: its
                `git config --local --list` holds no `remote.` key and no path, and is the same text under any ROOT.
push_to_remote  scratch clone from https://fixture.invalid/<name>.git, commit, push, remove the clone; the bare
                repository's config is not touched
                  no branch named                the commit continues the remote's default branch
                  a branch the remote has        the commit continues that branch from the remote's tip
                  a branch the remote lacks      the branch is created from the tip of the remote's default
                                                 branch, the commit is made on it, and it is pushed as a new
                                                 branch of the remote; the default branch does not move
```

```rust
// workspace.rs — a fixture workspace under <ROOT>/ws, built from literal inputs
pub struct WorkspaceBuilder<'t> { /* … */ }
impl<'t> WorkspaceBuilder<'t> {
    pub fn new(tree: &'t TempTree) -> Self;
    pub fn config(self, toml: &str) -> Self;               // .winter/config.toml, verbatim
    pub fn local_config(self, toml: &str) -> Self;         // .winter/config.local.toml
    pub fn state(self, toml: &str) -> Self;                // .winter/state.toml
    pub fn lock(self, toml: &str) -> Self;                 // .winter/config.lock
    pub fn file(self, relative: &str, content: &[u8]) -> Self;
    pub fn remote(self, name: &str, commits: Vec<CommitSpec>) -> Self;     // default branch "main"
    pub fn project(self, name: &str) -> Self;              // clone of remote <name> at projects/<name>
    pub fn env_worktree(self, env: &str, project: &str) -> Self;   // worktree <env>/<project>, branch <env>, from origin/main
    pub fn root_repository(self, gitignore: &str) -> Self; // the workspace directory is a git repository with one commit
    pub fn build(self) -> Result<Workspace, GitError>;
}
pub struct Workspace { pub root: PathBuf }
```

```text
build order   remotes -> files and .winter/ files -> projects -> env worktrees -> root repository commit
              the root commit holds every file present at that point that .gitignore does not exclude
no network    nothing the builder does resolves a host name
```
