# Invocation

How the binary starts the CLI of a nested workspace: which program runs, the environment it gets, the guards that refuse
before anything starts, and the verification that must hold before any mutating call. Every nested call of
[delegation.md](./delegation.md), [child-state.md](./child-state.md), and [teardown.md](./teardown.md) goes through this
leaf. The program port is that of
[child-processes.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/child-processes.md), with the
streaming, `stdin`, and `remove` that [reconciliation.md](../../workspace-bootstrap/spec/reconciliation.md), Commands,
adds to it; `RepoError` and `Termination` are those of
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
Domain. Reference locations are paths under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`.
Message wording is the binary's own (`decision:functional-parity`).

## Domain

```rust
// crates/core/src/programs.rs — the program port gains the nested CLI
pub(crate) enum Program { Git, Shell, Winter }
// Winter: the name "winter", looked up as Git is (child-processes.md, the program), but in the PATH the child
// receives: the call's `env` overlay when it sets PATH, else the snapshot's. So the shim on PATH picks the child
// root's own CLI, and a virtualenv's bin directory that the scrub removed is not searched.
// decision:parallel-workspaces/nested-workspaces/child-compatibility; reference: container.py:530-538,
// modules/workspace/internal/subprocess_nested_workspace_runner.py:23-37
```

```rust
// crates/core/src/nested.rs — engineering:nested-invocation-contract, engineering:child-authority-confinement
pub(crate) const CHAIN: &str = "WINTER_NESTED_CHAIN";

/// One per invocation, shared by every nested call it makes and across the threads of a fan-out.
pub(crate) struct Nesting<'p> {
    ports: &'p Ports<'p>,
    snapshot: &'p ProcessEnv,
    own_root: PathBuf,                     // this invocation's workspace root, resolved
    chain: Vec<PathBuf>,                   // parsed from the snapshot's WINTER_NESTED_CHAIN, outermost first
    verified: Mutex<HashSet<PathBuf>>,     // resolved roots a status read has verified in this process
}

impl Nesting<'_> {
    pub(crate) fn new<'p>(ports: &'p Ports<'p>, snapshot: &'p ProcessEnv, own_root: &Path) -> Nesting<'p>;
    /// Preflight, then a status read that verifies `root` and returns what it holds (child-state.md).
    pub(crate) fn state(&self, root: &Path) -> Result<NestedState, RepoError>;
    /// Preflight, then a status read only when `root` is not yet verified in this process.
    pub(crate) fn ensure_verified(&self, root: &Path) -> Result<(), RepoError>;
    /// ensure_verified, then `winter <args>` in `root`, streamed; the returncode as reconciliation.md gives it.
    pub(crate) fn run(&self, root: &Path, args: &[&OsStr], line: &mut dyn FnMut(&str)) -> Result<i32, RepoError>;
    /// Whether `root`'s committed config merged with its local overlay binds a service provider (teardown.md).
    pub(crate) fn binds_service(&self, root: &Path) -> Result<bool, RepoError>;
}

/// The snapshot minus the outer workspace's variables, as `remove` names and `env` overlay entries for a Call.
/// Also the environment of a nested project's `cmd` (reconciliation.md, Commands), which gets no chain.
pub(crate) fn scrubbed(snapshot: &ProcessEnv) -> Scrub;
pub(crate) struct Scrub { pub remove: Vec<OsString>, pub env: Vec<(OsString, OsString)> }
```

```rust
// crates/core/src/termination.rs — RepoMessage gains; each RepoError carries cwd = the nested root
OwnRootOnChain { own, root },      // "this winter resolved workspace <own>, which an enclosing nested call already
                                   //  runs in; refusing to run the nested workspace at <root> from it"
RootOnChain { root },              // "nested workspace at <root> is already being run by an enclosing winter call;
                                   //  refusing to re-enter it"
NoNestedConfig { root },           // "nested workspace at <root> has no .winter/config.toml; refusing to run winter
                                   //  there"
NestedStatusFailed { root },       // exit outside 0 and 1, or a signal; program "winter", subcommand "ws",
                                   // args ["status", "--json"], with the child's stderr
NestedStatusNotJson { root, cause },
NestedStatusShape { root, detail },   // detail names the field and what it held, child-state.md, Shape rules
NestedRootMismatch { root, reported: Option<String> },   // "winter in <root> resolved workspace <reported>, not the
                                                          //  nested workspace; refusing to run it"
NestedConfigUnreadable { path, cause },   // the child's config.toml or config.local.toml is not valid TOML
NestedNotStarted { command, root, cause },   // "`<command>` could not start in <root>"
```

## Interfaces

### Child environment

```text
# engineering:nested-invocation-contract, engineering:child-process-isolation
# reference: modules/workspace/nested_env.py:12-67
base        the git child environment of child-processes.md (WINTER_LAUNCH_TIME left out, GIT_SSH_COMMAND defaulted,
            TRACEPARENT and TRACESTATE as the snapshot holds them), without LANGUAGE and LC_ALL forced to C
remove      every name of the snapshot that starts "WINTER_", except WINTER_LOG_LEVEL and names starting "WINTER_OTEL_"
            VIRTUAL_ENV
env         PATH, when the snapshot holds both VIRTUAL_ENV and PATH: PATH split at ":" with every entry whose normalised
            path equals normalise(<VIRTUAL_ENV>/bin) left out, rejoined with ":"; normalise is lexical (".", "..",
            repeated and trailing "/"), no filesystem access
            WINTER_NESTED_CHAIN = the entries of the snapshot's chain, then own_root, joined with ":"; set for
            nested CLI calls only, never for a `cmd`
chain       the snapshot's WINTER_NESTED_CHAIN as bytes, split at ":", empty entries dropped; absent is empty
own_root    the workspace root of this invocation, resolved (symbolic links followed)
```

### Preflight

```text
# engineering:child-authority-confinement; reference: modules/workspace/nested_workspace_service.py:274-293
every nested call, the status read included, runs these in order before starting anything:
  1. own_root is an entry of chain                        -> OwnRootOnChain     # a misresolved child stops here,
                                                                                # so it cannot fan out to siblings
  2. resolve(root) is an entry of chain                   -> RootOnChain
  3. <root>/.winter/config.toml is not a regular file     -> NoNestedConfig
entries compare as byte strings to the resolved paths; a nested call never runs without all three passing
```

### Status read and verification

```text
# decision:parallel-workspaces/nested-workspaces/child-compatibility
# reference: modules/workspace/nested_workspace_service.py:172-181,267-313,
#            modules/workspace/internal/subprocess_nested_workspace_runner.py:13-37
call        Program::Winter, args ["ws", "status", "--json"], cwd root, stdin inherited as reconciliation.md's
            commands, stdout and stderr captured whole, timeout none, the child environment above
exit        Code(0) or Code(1) -> stdout is the document; any other code or a signal -> NestedStatusFailed
            the program cannot be started -> NestedNotStarted, command "winter ws status --json"
document    stdout not JSON (UTF-8 required)            -> NestedStatusNotJson
            not schema v1 (child-state.md, Shape rules) -> NestedStatusShape
            workspace.root_path absent, not a string, or resolve(root_path) != resolve(root) -> NestedRootMismatch
            any other shape fault                       -> NestedStatusShape
verified    a read that passes adds resolve(root) to `verified`; state() always reads, ensure_verified() reads only
            when resolve(root) is not in `verified`. The set is behind a mutex; two threads verifying one root may
            both read, and either result stands. Nothing is remembered across invocations
recursion   bounded only by the chain: no depth limit
```

### Mutating calls

```text
# engineering:child-authority-confinement
# reference: modules/workspace/nested_workspace_service.py:88-101,183-265,315-319
run(root, args)
  ensure_verified(root); its RepoError ends the call before anything starts
  Program::Winter, args as given (never through a shell), cwd root, stdin inherited, stdout and stderr joined on one
  pipe and delivered line by line as reconciliation.md, Commands, gives for `cmd`; timeout none
  -> the returncode: the exit code, or minus the signal number; NestedNotStarted when it cannot be started
  the caller reports CmdStarted before and CmdCompleted after, under the outer project's name
the calls this epic makes
  winter ws init                                                        delegation.md
  winter ws destroy <env> [--force] [--strict] [--no-provision-teardown]  teardown.md
  winter service down workspace                                         teardown.md
interrupt   the child runs in the binary's process group, so a terminal's SIGINT reaches it as well
            (child-processes.md, signals)
```
