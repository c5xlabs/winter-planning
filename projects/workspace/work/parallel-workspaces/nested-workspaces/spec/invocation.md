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
pub const CHAIN: &str = "WINTER_NESTED_CHAIN";
pub fn chain(snapshot: &ProcessEnv) -> Vec<PathBuf>;   // chain, Child environment
/// The environment of a nested CLI call (Child environment): scrubbed's remove and env, then WINTER_NESTED_CHAIN of
/// chain(snapshot) and own_root, then TRACEPARENT and TRACESTATE of tracer.child_context().
pub fn child_environment(snapshot: &ProcessEnv, own_root: &Path, tracer: &dyn CommandTracer) -> Scrub;

/// One per invocation, shared by every nested call it makes and across the threads of a fan-out.
pub struct Nesting<'p> {
    ports: &'p Ports<'p>,
    snapshot: &'p ProcessEnv,
    own_root: PathBuf,                     // this invocation's workspace root, resolved
    chain: Vec<PathBuf>,                   // parsed from the snapshot's WINTER_NESTED_CHAIN, outermost first
    verified: Mutex<HashSet<PathBuf>>,     // resolved roots a status read has verified in this process
}

impl Nesting<'_> {
    pub fn new<'p>(ports: &'p Ports<'p>, snapshot: &'p ProcessEnv, own_root: &Path) -> Nesting<'p>;
    /// Preflight, then a status read that verifies `root`, then the child's declared standalones present on disk;
    /// returns what it holds (child-state.md).
    pub fn state(&self, root: &Path) -> Result<NestedState, RepoError>;
    /// Preflight, then a status read only when `root` is not yet verified in this process.
    pub fn ensure_verified(&self, root: &Path) -> Result<(), RepoError>;
    /// ensure_verified, then `winter <args>` in `root`, streamed; the returncode as reconciliation.md gives it.
    pub fn run(&self, root: &Path, args: &[&OsStr], line: &mut dyn FnMut(&str)) -> Result<i32, RepoError>;
    /// Whether `root`'s committed config merged with its local overlay binds a service provider (teardown.md).
    pub fn binds_service(&self, root: &Path) -> Result<bool, RepoError>;
}

/// The snapshot minus the outer workspace's variables, as `remove` names and `env` overlay entries for a Call.
/// Also the environment of a nested project's `cmd` (reconciliation.md, Commands), which gets no chain.
pub(crate) fn scrubbed(snapshot: &ProcessEnv) -> Scrub;
pub struct Scrub { pub remove: Vec<OsString>, pub env: Vec<(OsString, OsString)> }
```

```rust
// crates/core/src/trace.rs — CommandTracer gains, beside command_started and operation
/// The W3C trace context a child process continues: (traceparent, tracestate), the context of the command span,
/// which parents every operation span, so it is the span active at any nested call. None with tracing off or the
/// command span not recording. The default body answers None, so NoTrace and the test tracers need no change.
fn child_context(&self) -> Option<(String, Option<String>)> { None }
// crates/cli/src/trace.rs — the binary's Tracer answers it by injecting the command span's context through
// opentelemetry_sdk's TraceContextPropagator into a carrier and reading "traceparent" and "tracestate" back
```

`scrubbed` and `Scrub` are this leaf's contract, given by `remove` and `env` of Child environment below without
`WINTER_NESTED_CHAIN`. `epic:parallel-workspaces/workspace-bootstrap` landed the scrub first, for the `cmd` of a nested
project (reconciliation.md, Commands), as the nested branch of the private `command_environment` of
`crates/core/src/bootstrap/checkout.rs`, whose PATH filter compares each entry's bytes with `<VIRTUAL_ENV>/bin` and does
not normalise. This epic moves that branch alone, the `WINTER_*` and `VIRTUAL_ENV` removal and the PATH filter, to
`nested::scrubbed` and `Scrub` in `crates/core/src/nested.rs`, with the lexical normalise of Child environment; no
second scrub remains. `command_environment` and `CommandEnvironment` stay in `checkout`, with the `COMMAND_VARIABLES`
overlay set on every `cmd` and the path of a project without `nested`; for a nested project it lays `scrubbed`'s
`remove` and `env` before that overlay, as reconciliation.md, Commands, orders them. `child_environment` takes no
`COMMAND_VARIABLES`: a nested CLI call is no `cmd`.

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

Visibility: `CHAIN`, `chain`, `child_environment`, `Scrub`, `Nesting`, and `Nesting`'s `new`, `state`,
`ensure_verified`, `run`, and `binds_service` are `pub` inside their private module and re-exported by name from
`lib.rs`, each module's names on a `pub use` line of their own, with every type their signatures name
(`context/architecture/changes.md`, A domain module, of `winter-workspace`), so the slice that lands each tests it
before a command calls it (`context/standards/testing.md`) and the gate finds nothing unused without an `allow`
(`context/standards/rust.md`). `chain` is re-exported as `nested_chain`, since
`epic:parallel-workspaces/stacked-history` re-exports its own `chain` as `restack_chain` and neither keeps the bare name
at the crate root; `run` and the other methods are reached through `Nesting` and add no name there. `Program` and
`CommandTracer` are re-exported already. Everything else here stays `pub(crate)`: `scrubbed` has its caller in
`checkout` from the change that moves it.

## Interfaces

### Child environment

```text
# engineering:nested-invocation-contract, engineering:child-process-isolation
# reference: modules/workspace/nested_env.py:12-67
base        the git child environment of child-processes.md (WINTER_LAUNCH_TIME left out, GIT_SSH_COMMAND defaulted),
            without LANGUAGE and LC_ALL forced to C
            TRACEPARENT, TRACESTATE: tracer.child_context() Some((p, s)) -> env TRACEPARENT = p, and TRACESTATE = s
            when s is Some, else TRACESTATE in remove, since an overlay cannot unset a name; so the child's command
            span is a child of the outer command span. None (tracing off) -> as the snapshot holds them
            (tracing.md; engineering:observability-isolation)
            the program port of child-processes.md is unchanged: it passes the snapshot's TRACEPARENT, and this
            call's env and remove replace it, as tracing.md leaves child propagation to the port's caller
            # reference: core/internal/local_subprocess_runner.py:187-190 writes the active context
            tracing.md = ../../../../delivered/parallel-workspaces/component-runtime/spec/tracing.md
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
