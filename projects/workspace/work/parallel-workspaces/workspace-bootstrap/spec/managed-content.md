# Managed content

The managed blocks Workspace owns in files it shares with the user: the workspace exclude file and its blocks, and the
eager safety block of `AGENTS.md` with the `CLAUDE.md` shim. `ws init` ([reconciliation.md](./reconciliation.md)),
`init` ([creation.md](./creation.md)), and `ws prune` ([prune.md](./prune.md)) call what is here. Every git call goes
through the program port of
[child-processes.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/child-processes.md); every file
write goes through the file port's `update`, which takes a lock on the file's directory and replaces the file by rename
([lock-and-state.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/lock-and-state.md), Write
integrity). Reference locations are paths under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `b57b020`.
Message wording is the binary's own (`decision:functional-parity`).

## Domain

```rust
// crates/core/src/managed_block.rs — pure functions over text; no port. Reference: modules/workspace/internal/managed_block.py
pub(crate) struct Markers { pub begin: String, pub end: String }
impl Markers {
    pub(crate) fn exclude(name: &str) -> Self;   // "# >>> <name> (managed by winter)", "# <<< <name>"
    pub(crate) fn markdown(name: &str) -> Self;  // "<!-- >>> <name> (managed by winter) -->", "<!-- <<< <name> -->"
}
pub(crate) enum Position { Append, Prepend }
// The block is `body` between the two markers. Lines are split on "\n"; the result ends with "\n".
pub(crate) fn replace_or_insert(content: &str, markers: &Markers, body: &[&str], position: Position) -> String;
pub(crate) fn strip(content: &str, markers: &Markers) -> String;
pub(crate) fn blocks(content: &str) -> Vec<(String, Vec<String>)>; // exclude-style blocks in file order: name, lines
pub(crate) enum Shape { Absent, One, Malformed }                    // Malformed: a begin without its end, or two begins
pub(crate) fn shape(content: &str, markers: &Markers) -> Shape;

// crates/core/src/workspace_exclude.rs — the workspace exclude file. engineering:workspace-exclude-isolation
pub(crate) enum Located { NotARepository, File(PathBuf) }
pub(crate) fn locate(programs: &dyn RunPrograms, tracer: &dyn CommandTracer, files: &dyn ReadFiles, root: &Path)
    -> Result<Located, GitFailure>;          // reads only; never writes git configuration
pub(crate) fn locate_for_write(programs: &dyn RunPrograms, tracer: &dyn CommandTracer, files: &dyn ReadFiles,
    root: &Path) -> Result<Located, GitFailure>;
pub(crate) fn write_block(ports: &Ports<'_>, root: &Path, name: &str, lines: &[String])
    -> Result<Option<PathBuf>, GitFailure>;  // Some(file) when the file changed; None when unchanged or no repository
pub(crate) fn finalize(ports: &Ports<'_>, root: &Path, mode: AdoptExtensions, eligible: &[Eligible], keep: &[String])
    -> Result<Finalized, GitFailure>;   // keep: block names left as they are; prune.md, reaggregate
pub(crate) struct Eligible { pub name: String, pub relative: String, pub extra: Vec<String> }
// extra: the extension lines of epic:agent-ready-operation/extension-management, which builds them; empty until then
pub(crate) enum Finalized { NotARepository, Unchanged, Updated { file: PathBuf, names: Vec<String> } }
// Updated: the eligible blocks changed. A change to the winter-config block alone is Unchanged, as is every run under
// adopt_extensions = "none": the reference writes that block on its own and never reports it

// crates/core/src/agents_md.rs — the eager safety block. product:managed-content-ownership
pub(crate) const SAFETY_BLOCK: &str = "winter-workspace/safety";
pub(crate) enum Placement { Created, Updated, Unchanged, Absent, Conflict(Conflict) }
pub(crate) enum Conflict { MalformedBlock { path: PathBuf }, NotUtf8 { path: PathBuf } }
pub(crate) enum Shim { Created, Present, Conflict { path: PathBuf } }
pub(crate) fn place(writer: &dyn WriteFiles, root: &Path) -> io::Result<Placement>;   // init: create or update
pub(crate) fn refresh(writer: &dyn WriteFiles, root: &Path) -> io::Result<Placement>; // ws init: update only
pub(crate) fn shim(writer: &dyn WriteFiles, root: &Path) -> io::Result<Shim>;

// Ports is the Ports of crates/core/src/pins.rs, which materialize takes: files, writer, programs, tracer. It moves to
// crates/core/src/ports.rs, re-exported from pins.rs and lib.rs under its present names, so bootstrap, prune, and pins
// share one type
```

```rust
// crates/core/src/files.rs — WriteFiles gains; OsFiles implements both with std::fs. MemoryFiles stays a ReadFiles
// alone: a test that writes runs OsFiles over a temporary tree, as the write tests of files.rs do
fn create_dir_all(&self, path: &Path) -> io::Result<()>;
// A symbolic link is removed as a link, never followed; a directory is removed with everything under it, links inside
// it removed as links; anything else is unlinked. A missing path is Ok. An error stops the removal and leaves what was
// not yet removed in place.
fn remove(&self, path: &Path) -> io::Result<()>;
```

### Ports across threads

```text
# for the fan-out of reconciliation.md, Fan-out; engineering:observability-isolation
ReadFiles, WriteFiles, RunPrograms, CommandTracer    gain Sync as a supertrait
crates/cli/src/trace.rs, Tracer                      state: RefCell<State<E>> becomes Mutex<State<E>>; each method
                                                     holds the lock for its own body alone
operation spans from a worker                        operation() opens the span with the command span as its explicit
                                                     parent, from the context State::Open holds, never from the
                                                     calling thread's current context; so a span a worker opens is a
                                                     child of the command span, as one the calling thread opens is.
                                                     The handle is opened and dropped on one thread
the command span                                     unchanged: conclude runs on the calling thread after every worker
                                                     has joined, and hands the span and the provider to trace-export
test doubles                                         those that hold a RefCell (support/spans.rs, support/in_process.rs,
                                                     git_operations.rs, locations.rs, repository_catalog.rs) hold a
                                                     Mutex; what they record and assert is unchanged
context/architecture/structure.md                    gains: bootstrap.rs starts scoped worker threads, which join
                                                     before reconcile returns, the one thread start in crates/core;
                                                     trace-export stays the one thread that outlives its caller
```

## Interfaces

### Managed block text

```text
# reference: modules/workspace/internal/managed_block.py:17-95
replace_or_insert(content, markers, body, position)
  lines := content split on "\n" ([] for "")
  begin found at i:
    its end found at j > i     -> lines[..i] + block + lines[j+1..]
    no end after i             -> lines[..i] + block          # to end of file, as the reference does
  absent, Append               -> lines, then "" when the last line is not blank, then block
  absent, Prepend              -> block, then "" when the first line is not blank, then lines
  result := lines joined with "\n", then "\n" when it does not end with one
  block := [begin, body..., end]
strip(content, markers)        -> the block and the one blank line directly above it removed; content unchanged when
                                  the begin marker is absent; a begin without end strips to end of file
blocks(content)                -> each line matching ^# >>> (.+?) \(managed by winter\)$ starts a block named by the
                                  group; its lines run to "# <<< <name>" or to end of file
markers are matched as whole lines, exactly; no trimming
```

### The workspace exclude file

```text
# engineering:workspace-exclude-isolation; reference: modules/workspace/internal/gitpython_workspace_exclude_locator.py:30-115
locate(root)
  <root>/.git absent (neither file nor directory)          -> NotARepository   # the root itself, no parent search
  g := resolve(`git rev-parse --absolute-git-dir` in root) ; a failure is GitFailure (GitOperation::ExcludeLocate)
  -> File(g/info/exclude)
locate_for_write(root)
  locate(root); NotARepository -> NotARepository
  c := resolve(root / `git rev-parse --git-common-dir`)
  g != c (a linked worktree):
    `git config --local --type=bool --get extensions.worktreeConfig` prints "true"   -> nothing
    otherwise, for key in core.bare, core.worktree:
      v := `git config --file <c>/config --get <key>`; exit 1 -> skip
      `git config --file <c>/config.worktree <key> <v>`, then `git config --file <c>/config --unset <key>`
      then `git config extensions.worktreeConfig true`
    `git config --worktree --get core.excludesFile` != g/info/exclude -> `git config --worktree core.excludesFile <g/info/exclude>`
  -> File(g/info/exclude)
# every read precedes its write, so a repeated or sibling run that finds the settings in place writes no configuration
# a `git config --get` exit of 1 is "unset"; any other failure is GitFailure (GitOperation::Config)
# GIT_DIR relocation of the workspace is not supported, as in the reference
```

| Block                        | Lines                                         | Written by                                       | Stripped by                        |
| ---------------------------- | --------------------------------------------- | ------------------------------------------------ | ---------------------------------- |
| `winter-dir/projects`        | `/projects/`                                  | `ws init`, projects target                       | never                              |
| `winter-workspace/artifacts` | `/.winter/logs/`                              | `ws init`, projects target                       | never                              |
| `winter-workspace/workspace-skills` | the six vendor skill globs of `prefix` | `ws init`, projects target                    | never                              |
| `winter-dir/<env>`           | `/<env>/`                                     | `epic:parallel-workspaces/environment-lifecycle` | `ws destroy`, that epic            |
| `winter-config`              | `.winter/config/**/*.local.*`                 | `finalize`, always, whatever the mode            | never                              |
| `<name>`, no `/` in it       | `/<relative>/`, then the eligible's `extra`   | `finalize`, per eligible                         | `finalize`, when not eligible      |

```text
write_block(root, name, lines)   # reference: modules/workspace/init_service.py:800-841
  locate_for_write(root); NotARepository -> None
  update(file): replace_or_insert(existing or "", exclude(name), lines, Append); unchanged -> no write, None
  -> Some(file)

finalize(root, mode, eligible, keep)   # reference: modules/workspace/extension_exclude_service.py:66-170
  locate_for_write(root); NotARepository -> NotARepository
  one update of the file, under its lock:
    base := replace_or_insert(existing, exclude("winter-config"), [".winter/config/**/*.local.*"], Append)
    mode == None -> content := base; nothing else is touched     # adopt_extensions = "none"
    otherwise:
      names := the names of eligible
      content := base, with every block stripped whose name matches ^[^/]+$, is not "winter-config", is not in names,
                 and is not in keep
      for each eligible in the given order: replace_or_insert(content, exclude(name), ["/<relative>/", extra...], Append)
    the file is written when content differs from existing
  mode != None and content differs from base -> Updated { file, names: names sorted }; otherwise Unchanged
# one read-modify-write of the whole file, so a concurrent writer's block is either read and kept or written after
```

`finalize` is the only remover of `<name>` blocks; `winter-dir/…` and `winter-workspace/…` names hold a `/` and are
never stripped by it. Removing the last standalone calls `finalize` with no eligible, which strips that block alone
(`product:managed-content-ownership`).

### The eager safety block and the shim

```text
# product:managed-content-ownership, product:create-unforked-workspace;
# decision:parallel-workspaces/workspace-bootstrap/creation-ownership
# body := eager_text() of epic:agent-ready-operation/context-navigation, split on "\n", its final empty line dropped
# markers := markdown("winter-workspace/safety"); the file is <root>/AGENTS.md
place(root)                                    # one update of AGENTS.md
  absent                         -> block then "\n"                                    Created
  not UTF-8                      -> no write                                           Conflict(NotUtf8)
  shape Malformed                -> no write                                           Conflict(MalformedBlock)
  shape One                      -> replace_or_insert(.., Prepend): unchanged -> Unchanged, else Updated
  shape Absent                   -> replace_or_insert(.., Prepend)                     Updated
refresh(root)                                  # as place, except that it decides Absent first:
  AGENTS.md absent, or its bytes do not hold the begin marker's bytes -> no write       Absent
  # so a file that is not UTF-8 and holds no block is no conflict; only a file that holds the block is ever written
shim(root)                                     # <root>/CLAUDE.md, one update
  absent                         -> "@AGENTS.md\n"                                      Created
  a line equal to "@AGENTS.md"   -> no write                                            Present
  otherwise                      -> no write                                            Conflict
# user text outside the markers is never rewritten: replace_or_insert keeps every other line byte for byte, and a
# conflict writes nothing. Prepend puts the safety rules first, where a harness that truncates reads them.
```
