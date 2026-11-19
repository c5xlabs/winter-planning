# Child processes

The port through which the library starts a program and waits for it, and the environment a git child gets. It takes the
three duties [command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md),
Process inputs, and [tracing.md](../../../../delivered/parallel-workspaces/component-runtime/spec/tracing.md) place on
the first command that starts a child process, which is `ws worktrees --status`
([location-listing.md](./location-listing.md)). Later commands that start git use the same port and add what their own
programs need; a child of another kind, such as a nested workspace's, gets the environment its own epic states
(`engineering:parallel-workspaces/nested-workspaces/child-process-isolation`). Reference locations are paths under
`tools/winter-cli/src/winter_cli/` of `winter-workspace` at `b57b020`; GitPython is the 3.1.46 the reference tool runs
git through.

## Interfaces

```text
# engineering:catalog-effect-separation; engineering:parallel-workspaces/component-runtime/testable-boundaries,
# engineering:parallel-workspaces/component-runtime/library-output-boundary
a git child's environment, built once per invocation from the environment snapshot, never from the live process
environment, so the variables the binary removes from its own environment for the exporter reach the child
  every name of the snapshot, with its value, bytes as given        except:
  WINTER_LAUNCH_TIME          left out                              reference: cli.py:365-377
  GIT_SSH_COMMAND             when the snapshot has no such name, set to
                              ssh -o ConnectTimeout=10 -o ServerAliveInterval=30 -o ServerAliveCountMax=3
                              a value present, the empty one included, is kept
                              reference: core/ssh_keepalives.py:12-25
  LANGUAGE, LC_ALL            set to C for git, replacing any value  reference: GitPython git/cmd.py:1243-1246
  TRACEPARENT, TRACESTATE     passed as the snapshot holds them; nothing is put in their place for git
the program
  "git" is looked up in the PATH of the snapshot, entry by entry, as execvp does; the first executable regular
  file named git is run; an empty entry is the call's working directory. PATH unset: /bin:/usr/bin, Python's
  os.defpath, which the reference's subprocess searches.
  PATH is the one variable the binary itself reads to start a program; HOME is not read. The Process inputs table of
  command-surface.md is completed by this row:
    PATH    The directories searched for git when a command starts it; also passed to the child.
cwd         the directory the call names
stdin       the null device                                         reference: GitPython git/cmd.py:1284
stdout      captured whole; stderr captured whole; neither reaches the binary's own streams
timeout     none for these calls; WINTER_GIT_TIMEOUT_S bounds remote calls, which the epics that fetch and push
            specify
signals     the child runs in the binary's process group, so a terminal's SIGINT reaches it as well
            (../../../../delivered/parallel-workspaces/component-runtime/spec/runtime.md, Interrupts and signals)
a program that cannot be started, git not found included
            InternalError::Unexpected, exit 1. The reference fails the same way, earlier: GitPython runs
            `git version` when it is imported and raises ImportError without git (git/__init__.py:295-298,
            git/cmd.py:785-789,824), and the module that lists environments imports it
            (modules/workspace/internal/read_workspace_repository.py:5), so its `ws worktrees` fails without git
            even without --status. The binary starts git only for --status; that difference is an artifact of a
            Python library under decision:functional-parity
```

Spans: the port starts no span of its own. A command wraps the calls of one operation in an operation span through the
tracer port below; `ws worktrees --status` opens one per probe ([location-listing.md](./location-listing.md), tracing).

## Domain

```rust
// crates/core/src/programs.rs — the only module of the library that starts a process; the architecture check of
// ../../../../delivered/parallel-workspaces/component-runtime/spec/repository-layout.md, Library rules, adds
// `process::Command` to its forbidden tokens everywhere else (files.rs keeps its std::process::id)
pub(crate) enum Program { Git }
pub(crate) struct Call<'a> { pub program: Program, pub args: &'a [&'a OsStr], pub cwd: &'a Path }
pub(crate) struct Finished { pub exit: Exit, pub stdout: Vec<u8>, pub stderr: Vec<u8> }
pub(crate) enum Exit { Code(i32), Signal(i32) }      // RepoError.exit: a code, or minus the signal number

pub(crate) trait RunPrograms {
    // Starts the program and waits for it. Err only when it cannot be started.
    fn run(&self, call: &Call<'_>) -> std::io::Result<Finished>;
}
// The programs of the operating system; built by `run` beside OsFiles, from the snapshot.
pub(crate) struct OsPrograms { /* the child environment above, and the PATH entries */ }
impl OsPrograms { pub(crate) fn new(snapshot: &ProcessEnv) -> Self; }
// The lookup of a program on a PATH value, pure: the first entry where `executable` holds; None when none does.
pub(crate) fn find_program(name: &str, path: Option<&OsStr>, cwd: &Path, executable: &dyn Fn(&Path) -> bool)
    -> Option<PathBuf>;

// crates/core/src/context.rs — Context gains; run_with fills both from the Invocation
pub programs: &'a dyn RunPrograms,
pub tracer: &'a dyn CommandTracer,             // the invocation's tracer, which today stops in run_with

// crates/core/src/trace.rs — CommandTracer gains, with the first operation span
// tracing.md, The library's part. Opens a child of the command span; it ends when the handle drops. With tracing
// off, or no command span recording, it records nothing. The default body returns a span that records nothing, so
// NoTrace and the test scaffold's tracer need no change; the binary's Tracer overrides it.
fn operation(&self, name: &'static str, attributes: &[(&'static str, &str)]) -> Box<dyn OperationSpan + '_> { … }
pub trait OperationSpan { fn failed(&mut self, error_type: &'static str); }
```

A component test runs the real port inside the isolated tree, whose `PATH` puts its stub programs first; a unit test of
`locations::list` hands it a `RunPrograms` of its own
([test-harness.md](../../../../delivered/parallel-workspaces/component-runtime/spec/test-harness.md), The isolated
tree).
