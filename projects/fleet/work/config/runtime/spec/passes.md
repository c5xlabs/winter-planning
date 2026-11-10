# Passes on the runner

Each pass applies [lifetimes.md](./lifetimes.md) §Evaluated at the start of a pass.

## System

Where a runner's configuration is frozen today, and what replaces each:

| Frozen today                                      | Where                                                                                                                                                                                | Becomes                                                                                                                                                                                       |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| The file, read once                               | `runner/host_config.py::HostConfig.load`                                                                                                                                             | `boot` keys from the file; every other key resolved from the store's entries                                                                                                                  |
| The loop's context, cached for the process's life | `runner/loop_wiring.py::RunnerStepper._context`; the runner prompt in `LoopWiring.of`                                                                                                | Rebuilt at the first tick after the runner's or host's revision moves (§A runner's tick)                                                                                                      |
| The tick interval, fixed into the timer thread    | `BZ_RUNNER_TICK_SECONDS` (`runner/cli/runtime.py`); `PeriodicDriver`                                                                                                                 | `tick_seconds`, evaluated before each wait (§The loop's interval)                                                                                                                             |
| Each runner's object graph                        | `runner/composition.py`: harness adapters and alias tables, subscription samplers, trace settings, the worker telemetry plan                                                         | Adapters, telemetry plan, and health probes take settings at the call; samplers and renewers rebuilt per record by the host tick; trace settings read per sweep (§The objects settings reach) |
| The set of runners                                | One `_RunnerThreads` per `[[runner]]` at boot (`runner/cli/runtime.py`): the tick driver, the trace-sweep driver where tracing is on, the spawner executor (`runner/composition.py`) | Reconciled by the host tick ([host-records.md](./host-records.md))                                                                                                                            |

| Pass            | Lock today                                                    | Evaluates at its start                                                                   |
| --------------- | ------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| A runner's tick | `RunnerStepper._tick_lock` (`runner/loop_wiring.py`)          | The runner's and the host's revisions                                                    |
| A trace sweep   | `RunnerStepper._trace_lock`                                   | The same revisions                                                                       |
| The host tick   | `HostPassStepper`'s `StepLock` (`runner/loop/step_passes.py`) | The host revision and every hosted runner, workspace, and subscription record's revision |

A step inside a pass (the transcript drain, retention, the context sample) reads its pass's snapshot. A stepped loop
(`loop = "stepped"`, `bzh:steppable-loop`) evaluates at the start of each stepped pass, so a test that writes a setting
and steps the tick sees it take hold in that step.

## Interfaces

### The loop's interval

```python
# runner/loop_wiring.py, runner/loop/periodic_pass_driver.py                 product:tick-interval-setting
class PeriodicDriver:
    def __init__(self, config: RunnerConfig, *, interval: Callable[[], float], process_graph: RunnerProcess,
                 stepper: RunnerStepper | None = None, wake: threading.Event) -> None: ...

class PeriodicPassDriver:
    def __init__(self, run_pass: Callable[[], None], *, name: str, label: str, log: BoundLogger,
                 interval: Callable[[], float], wake: threading.Event, jitter_seconds: float | None = None,
                 stop_timeout_seconds: float = STOP_TIMEOUT_SECONDS) -> None: ...

    def _run(self) -> None:
        while not self._stop.is_set():
            started = clock.now()
            self._run_pass()                    # a write never interrupts a running pass
            while True:
                due = started + self._interval()  # from the last snapshot the pass took; jitter drawn as today
                if clock.now() >= due or self._wait_until(due):  # _wait_until returns False when woken
                    break                       # woken: re-evaluate the interval, counted from the pass's start
```

| Driver                 | `interval()` reads         |
| ---------------------- | -------------------------- |
| A runner's tick        | `tick_seconds`, host-level |
| The host tick          | `tick_seconds`             |
| A runner's trace sweep | `tracing.sweep_seconds`    |

A committed write of `tick_seconds` or `tracing.sweep_seconds` sets the drivers' `wake` event; a woken driver runs at
once if the new due time has passed, or waits out the rest.

### Sessions

```python
# product:session-settings
@dataclass(frozen=True)
class SessionSettings:
    # Built once per launch from the tick's snapshot: every `session` key a worker is handed. The adapter's launch
    # composes the worker's command, environment, and settings from this and from nothing else; a worker never
    # re-reads a setting, a `credentials_path` included.
    harness: HarnessSections            # harness settings, harness.config_dir, harness.autonomy
    model_aliases: ...                  # models.*, effort.*, opencode.models.*, opencode.effort.*
    worker_env: ...                     # worker.env_passthrough, worker.path_prepend, worker.shell
    tools: ...                          # tools.env_passthrough, tools.path_prepend
    runner_prompt: str | None
    workspace_prompt: str | None
    transcripts_root: Path
    telemetry: ...                      # tracing.harness_telemetry, worker_programs, worker_program_services
    host_settings_revision: int
    runner_settings_revision: int
```

```python
# wire/lease.py
class LeaseView(WireModel):
    ...
    host_settings_revision: int | None      # from the lease's latest lease_spawns row; null before the revision
    runner_settings_revision: int | None
```

- Each spawn writes the two revisions and `transcripts_root` to its `lease_spawns` row
  ([runner-store.md](./runner-store.md) §Data). A resume after a restart, or a respawn after a retry, is a new spawn
  under the settings in force when it relaunches.
- A session's transcript is read from the `transcripts_root` on its spawn row, so moving `transcripts_root` loses no
  running session's transcript.

### The objects settings reach

| Object (`runner/composition.py`)     | Under this epic                                                                                                |
| ------------------------------------ | -------------------------------------------------------------------------------------------------------------- |
| Harness registry and adapters        | Built once, never rebuilt; holds no setting. `launch(session: SessionSettings, ...)`                           |
| Worker telemetry plan                | Built once; takes the session's telemetry keys at launch                                                       |
| Harness health probes                | Built once; `probe(binary: Path)`                                                                              |
| Harness version cache                | Keyed by binary path: a moved `claude_code.binary` is probed at the next health refresh, an unmoved one is not |
| The loop's hub client                | Keyed by its timeout                                                                                           |
| An adapter whose harness is disabled | Stays built; the harness stops being advertised                                                                |

## Flows

### A runner's tick

```python
# runner/loop_wiring.py                                     product:tick-lived-settings, product:runner-settings-all-live
class RunnerStepper:
    def _context(self) -> LoopContext:                       # called holding _tick_lock
        revisions = self._reader.revisions()
        if revisions == self._built_at:
            return self._ctx                                 # unchanged: reuse the last tick's context
        config = self._reader.resolve(revisions)
        wiring = LoopWiring.of(config, broker=...)           # resolves the runner prompt
        ctx = wiring.context(hub, self._graph, sweep_worker_scratch=False, tracer=...)  # settings as data only
        old, self._ctx, self._built_at = self._ctx, ctx, revisions
        if old is not None:
            old.close()                                      # after the swap, still under the lock: its hub client
                                                             # is never closed under a running tick, which
                                                             # close() does not guard against today
        return ctx

    def _tick(self) -> None:
        ctx = self._context()
        self._in_force.record(self._built_at, at=clock.now())   # runner-api.md §Views
        tick(ctx)                                            # registers, then claims, from the snapshot
```

- The worker-scratch sweep runs once, at the daemon's start, never on a rebuild, since a rebuild can race a spawn.
- Registration declares the snapshot's name, agents, gates, projects, bindings, supported models, and enabled harnesses,
  so the hub's queue answers a write from the first tick after it.

Lowering a value stops nothing that runs (`engineering:running-session-undisturbed`):

| Change                              | Effect                                                                                                                                                     |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `agents` below the running sessions | No claim until the running count is under it; no session stopped                                                                                           |
| A harness disabled                  | Not advertised from the next registration; its sessions finish                                                                                             |
| A project removed from `projects[]` | No further claim; a held chunk finishes under the binding it was claimed through (`execution/hosts.md` §Bindings)                                          |
| A binding's workspace retired       | The runner keeps its other bindings and claims nothing through that one; the hub flags the binding unreported, as today (`hub/domain/runners/bindings.py`) |
| The runner's subscription retired   | The runner runs as though it named none ([host-records.md](./host-records.md) §Domain)                                                                     |

Today a binding naming an undeclared workspace halts the runner before it registers; under this epic only a write is
refused. A write binding a project to a workspace the host never declared, or to a retired one, fails validation as the
file would at boot, and since every write validates the runner's whole configuration
([runner-store.md](./runner-store.md) §A write), a runner left bound to a retired workspace has every later write
refused until one fixes the binding. `execution/hosts.md` §Bindings changes to say both (`engineering:context-rules`).

### The trace sweep

The lease trace sweep takes `tracing.sweep_seconds`, `settle_seconds`, `max_lag_seconds`, `batch_limit`, and
`replay_max_window` from the snapshot at each sweep's start; its driver's `interval()` is that snapshot's
`tracing.sweep_seconds`.

### The host tick

```python
# runner/loop/step_passes.py — before the tick's existing work (reporting, sampling, renewing, RetiredRunnerWatch)
def host_tick(host: HostProcess) -> None:
    snapshot = host.reader.revisions()
    host.runners.reconcile(snapshot)          # 1. start each hosted runner without threads; drain each retired one
                                              #    (host-records.md §Flows)
    if snapshot.workspaces_moved:             # 2. any active workspace record's revision moved
        host.workspaces.set(WorkspaceRegistry.build(
            snapshot.active_workspaces,
            keep=host.workspaces.unmoved(snapshot),   # an unmoved workspace keeps its provider
            locks=host.workspace_locks,               # every workspace keeps its lock, by workspace_id
        ))                                    # one assignment; the registry may be empty
    for sub in snapshot.subscriptions_moved:  # 3. sampler and renewer rebuilt from the record; cadence state is in
        host.rebuild_sampler_and_renewer(sub) #    the store (external_usage_samples, renewal claims), so nothing is lost
    report = host_report(snapshot)            # 4. built per tick, not the functools.partial bound at composition, so a
    ...                                       #    created or retired workspace or subscription reaches that report
```

```python
# runner/environments/workspaces.py
class HostWorkspaces:
    # The one holder every reader of the host's workspaces reads at use: each runner's process and loop context, the
    # routes (app.state.workspaces, runner/app.py), the health probes over the spawn roots (runner/composition.py).
    # None holds a WorkspaceRegistry handed to it at composition.
    def current(self) -> WorkspaceRegistry: ...
    def set(self, registry: WorkspaceRegistry) -> None: ...
```

- Locks carried by `workspace_id` mean an acquisition through a replaced provider and one through its replacement take
  the same lock; no environment is handed to two claims (`bzh:environment-held-once`).
- A replaced provider holds nothing to close (`runner/environments/internal/basic_provider.py`, `winter_provider.py`);
  an acquisition already running under it finishes under it. A held environment stays held, since what is held is read
  from the store.
- A host with no active workspace serves no binding; its runners claim nothing until one is created.

## Open questions

- `SessionSettings`' field names and grouping: the spec settles what it holds, not its shape. Engineering.
- How the wake event reaches the drivers (a constructor argument above, or a holder the write path signals): the spec
  settles only that a committed interval write wakes them. Engineering.
