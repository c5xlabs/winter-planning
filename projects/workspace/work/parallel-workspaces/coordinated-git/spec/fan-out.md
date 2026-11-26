# Fan-out

How a command runs per-repository work in parallel: the bounds, the grouping of worktrees that share one Git store, the
order events and documents take, and how a worker reports. Every command that runs repository tasks at once uses
`fan_out`; a command not listed under Bounds runs its targets one at a time. The program port is that of
[child-processes.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/child-processes.md), whose
`RunPrograms`, `CommandTracer`, and `ReadFiles` are `Sync`; remote calls keep the timeout and retries of
[standalone-pins.md](../../../../delivered/parallel-workspaces/reproducible-inputs/spec/standalone-pins.md), Remote
calls. Reference locations are paths under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`.

## Domain

```rust
// crates/core/src/fan_out.rs — engineering:bounded-fan-out
pub(crate) const REMOTE_TASKS: usize = 4;   // reference: modules/workspace/internal/git_ops_service.py:21-24
pub(crate) const STATUS_PROBES: usize = 8;  // reference: modules/workspace/env_status_service.py:25-32

/// One unit of work. Tasks of one group run one after another on one worker; groups run at once up to the bound.
pub(crate) struct Group<T> { pub key: String, pub tasks: Vec<T> }

/// What a worker sends the coordinating thread, in the order it happens.
pub(crate) enum Message<R> {
    Line(Stream, String),   // a complete diagnostic line the worker's Diagnostics wrote; Stream::Err only
    Done(usize, R),         // the result of task <index>, where index is its position in the flattened input
}

/// Runs `work` for every task of every group with at most `bound` groups in flight, on std::thread::scope
/// workers. `on_message` runs on the calling thread, which owns stdout and stderr: it writes each line and each
/// event as it arrives. Returns the results in input order, whatever the completion order.
pub(crate) fn fan_out<T: Sync, R: Send>(
    bound: usize,
    groups: Vec<Group<T>>,
    work: &(dyn Fn(&T, &mut Diagnostics<'_>) -> R + Sync),
    on_message: &mut dyn FnMut(&Message<R>) -> Result<(), Termination>,
) -> Result<Vec<R>, Termination>;
```

## Interfaces

### Bounds

```text
# engineering:bounded-fan-out
REMOTE_TASKS (4)  ws fetch, ws pull, ws merge, ws status --fetch (its fetch), and the clones and fetches of
                  ws init; one group per project repository (its worktrees, serial) and one per standalone
STATUS_PROBES (8) ws status worktree probes of one environment, and the nested reads of
                  ../../nested-workspaces/spec/child-state.md; one task per worktree, no grouping
serial            ws push, ws connect, ws disconnect, ws diff, and every command that moves a pointer or removes
                  files (checkout, reset, clean, restack, destroy, prune): one target at a time, in selection
                  order, on the calling thread; fan_out is not used
the bound counts groups in flight, so concurrent git processes never exceed it: each worker runs one git call
at a time
```

### Shared stores

```text
# reference: modules/workspace/workspace_sync_service.py:276-324,382-412, workspace_merge_service.py:162-200
a project's worktrees share one Git store: its group first runs the project's one refresh, where the command
refreshes, then each selected worktree of the project in selection order
the refresh runs once per project per invocation, never once per worktree
two groups never touch one store: a standalone is its own store, and a project's source checkout belongs to its
project's group
```

### Order

```text
events     written by on_message as each Done arrives, so they follow completion order across groups and
           selection order within a group; a parity scenario compares them per target, not as one stream
lines      a worker's Diagnostics writes into a buffer that sends each complete line as Message::Line; the
           coordinating thread writes it to stderr when it arrives, so a retry notice or warning appears while its
           task still runs, and lines of two tasks never interleave within a line
documents  a document written after the fan-out lists targets in configuration order: environments in their
           rule's order (target-patterns.md), worktrees in merged project order, standalones in merged order
```

### Failure

```text
a task returns its own outcome, a failure included; one task's failure never stops another task or group
an Err from on_message (stdout cannot be written) stops the fan-out: no new task starts, the running ones finish,
their results are discarded, and fan_out returns that Termination
a panic in a worker is InternalError::Panic for the whole invocation, after the other workers finish
an interrupt ends the process as runtime.md's handler does for a command that does not defer; children in the
process group receive the same SIGINT
runtime.md = ../../../../delivered/parallel-workspaces/component-runtime/spec/runtime.md
```
