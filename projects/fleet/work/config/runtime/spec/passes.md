# Passes on the runner

Each pass applies [lifetimes.md](./lifetimes.md) §Evaluated at the start of a pass.

## System

Where a runner's configuration is frozen today, and what replaces each:

| Frozen today                                      | Where                                                                                                                                                                                | Becomes                                                                                                                                                                           |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| The file, read once                               | `runner/host_config.py::HostConfig.load`                                                                                                                                             | `boot` keys from the file; every other key resolved from the store's entries                                                                                                      |
| The loop's context, cached for the process's life | `runner/loop_wiring.py::RunnerStepper._context`; the runner prompt in `LoopWiring.of`                                                                                                | Rebuilt at the first tick after the runner's or host's revision moves (§A runner's tick)                                                                                          |
| The tick interval, fixed into the timer thread    | `BZ_RUNNER_TICK_SECONDS` (`runner/cli/runtime.py`); `PeriodicDriver`                                                                                                                 | `tick_seconds`, evaluated before each wait (§The loop's interval)                                                                                                                 |
| Each runner's object graph                        | `runner/composition.py`: harness adapters and alias tables, subscription samplers, trace settings, the worker telemetry plan                                                         | Harness set rebuilt when its keys move (§The harness set); samplers and renewers rebuilt per record by the host tick; trace settings read per sweep (§The objects settings reach) |
| The harness bindings and bundle                   | `runner/harness/wiring.py::select_harness_bindings`, `publish_harness_bundle`, `build_production_harnesses`, called once per runner at composition                                   | Part of the harness set (§The harness set)                                                                                                                                        |
| The set of runners                                | One `_RunnerThreads` per `[[runner]]` at boot (`runner/cli/runtime.py`): the tick driver, the trace-sweep driver where tracing is on, the spawner executor (`runner/composition.py`) | Reconciled by the host tick ([host-records.md](./host-records.md))                                                                                                                |

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
            self._run_pass()                    # a write never interrupts a running pass
            finished = clock.now()              # counted from the pass's end, as PeriodicDriver._run waits today
            while not self._stop.is_set():
                due = finished + self._interval()   # from the last snapshot the pass took; jitter drawn as today
                if clock.now() >= due:
                    break
                woken = self._wake.wait_until(due, stop=self._stop)  # returns early when woken or stopped
                if not woken:
                    break                       # due: run the next pass
                # woken: re-evaluate the interval, still counted from the pass's end
```

- An unedited host waits exactly as it does today: the full interval after each pass ends, so a long pass is never
  followed at once by the next.
- Each driver holds its own wake signal (`WakeSignals.for_driver(name)`); a committed interval write sets every
  driver's, so no driver consumes another's wake. A stop ends the wait at once.

| Driver                 | `interval()` reads         |
| ---------------------- | -------------------------- |
| A runner's tick        | `tick_seconds`, host-level |
| The host tick          | `tick_seconds`             |
| A runner's trace sweep | `tracing.sweep_seconds`    |

A committed write of `tick_seconds` or `tracing.sweep_seconds` sets every driver's wake signal; a woken driver runs at
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

| Object (`runner/composition.py`) | Under this epic                                                                                  |
| -------------------------------- | ------------------------------------------------------------------------------------------------ |
| The harness set                  | Built from the `session` keys it reads and rebuilt when they move (§The harness set)             |
| Worker telemetry plan            | Part of the harness set                                                                          |
| Harness health probes            | Part of the harness set                                                                          |
| Harness version cache            | Keyed by binary path and the binary's size and modification time: a moved binary is probed again |
| The loop's hub client            | Keyed by its timeout                                                                             |
| A harness disabled               | Its binding stays in the set for the sessions it launched; the harness stops being advertised    |

### The harness set

Today a runner's harnesses hold settings. At composition `select_harness_bindings` runs each version-chosen binary's
`--version` once to pick its binding; `publish_harness_bundle` publishes the bundle from `harness.config_dir`,
`harness.autonomy`, the harness sections, and that choice; `build_production_harnesses` assembles every enabled binding
from its section and those shared inputs. None is built again.

```python
# runner/harness/harness_set.py                                product:session-settings, product:session-lifetime
@dataclass(frozen=True)
class HarnessSet:
    key: HarnessSetKey                   # every `session` key the set reads: the harness sections (binaries
                                         # included), harness.config_dir, harness.autonomy, the worker environment,
                                         # and the telemetry keys; plus each binary's probed version
    selection: BindingSelection
    bundle: BundleSnapshot | None        # the snapshot path itself, never the `current` link
    registry: HarnessRegistry
    probes: Mapping[str, IHarnessHealthProbe]
    telemetry: HarnessTelemetryPlan

class HarnessSets:
    # One per runner. current() answers the set for the tick's snapshot, building one when its key differs from the
    # last set's: the version probe through the version cache, the bundle published, the bindings assembled. A
    # snapshot whose key is unchanged reuses the last set, so a write to a `pass` key builds nothing.
    def current(self, settings: RunnerConfig) -> HarnessSet: ...
    # A launch takes the current set and holds it for its session; supervising that session (its transcript, usage,
    # interrupt, and stop) goes through the set it launched with. A set no longer current is dropped when its last
    # session ends.
    def launch(self, session: SessionSettings) -> HarnessSet: ...
```

- A republish never changes a snapshot a running session launched from. Snapshots are content-hashed directories that
  are never rewritten, and only the `current` link moves, atomically (`runner/harness/internal/bundle_publisher.py`).
  Nothing deletes a snapshot today. Anything that later prunes snapshots keeps every one a live harness set holds.
- An unchanged bundle publishes to the snapshot it already has, so rebuilding a set for some other key reuses it.
- Verbs that compose bindings without serving (`runner/cli/harness.py`, `runner/cli/transcript.py`) keep selecting
  bindings and reading `current` as they do today.

## Flows

### A runner's tick

```python
# runner/loop_wiring.py                                     product:tick-lived-settings, product:runner-settings-all-live
class RunnerStepper:
    def _context(self) -> LoopContext:                       # called by the tick and the trace sweep, as today
        with self._build:                                    # the lock that guards the build today
            revisions = self._reader.revisions()
            if revisions == self._built_at:
                return self._ctx                             # unchanged: reuse the last context
            config = self._reader.resolve(revisions)
            wiring = LoopWiring.of(config, broker=...)       # resolves the runner prompt; may raise (§A rebuild that fails)
            client = self._client_for(config)               # the client lives on the stepper, keyed by its timeout:
                                                             # reused unless the timeout moved
            ctx = wiring.context(client, self._graph, sweep_worker_scratch=False, tracer=...)  # settings as data only
            self._ctx, self._built_at = ctx, revisions
            return ctx

    def _client_for(self, config: RunnerConfig) -> httpx2.Client:
        # A moved timeout builds a new client. The old one is closed only once neither the tick nor the trace sweep
        # can hold it: after both step locks are next idle, as request() takes both today. Closing it under _build
        # alone could close it under a running sweep.
        ...

    def _tick(self) -> None:
        ctx = self._context()
        self._in_force.record(self._built_at, at=clock.now())   # runner-api.md §Views
        tick(ctx)                                            # registers, then claims, from the snapshot
```

- The worker-scratch sweep runs once, at the daemon's start, never on a rebuild, since a rebuild can race a spawn.
- A rebuild that fails keeps the runner on its last good context and harness set (§A rebuild that fails).
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

### A rebuild that fails

A write is validated as a boot validates the file ([runner-store.md](./runner-store.md) §A write), but the world it
names can change afterwards. A prompt file can be deleted, or a binary removed, and `LoopWiring.of` raises at startup
for a missing prompt file today (`runner/loop_wiring.py`). When a rebuild raises:

- The runner keeps ticking on the context and harness set it last built. It claims and launches under them as before.
- The `in_force` view names the revision that failed and the error ([runner-api.md](./runner-api.md) §Views), and the
  runner logs `settings_rebuild_failed` once per failing revision, with the error.
- Each later tick tries the rebuild again, so a fix on disk takes hold without another write. Once one succeeds, the
  view and the log say so.
- At startup there is no last good context, so the runner fails to start as it does today.

### Bindings and workspaces

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
- How the write path reaches `WakeSignals` (a holder on the host process, or a callback the store's write runs after
  commit): the spec settles only that a committed interval write sets every driver's signal. Engineering.
