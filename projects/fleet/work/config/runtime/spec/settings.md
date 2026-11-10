# Settings and their lifetimes

## Domain

```python
# foundation/config_keys.py
class Lifetime(StrEnum):
    # Loaded once at the start of each pass of the loop that uses it, as one snapshot of every entry that pass
    # reads, and held until the pass ends. A hub request is a pass: it loads when it arrives. Nothing is re-read
    # mid-pass; a write committed during a pass reaches the next one.            product:pass-lifetime
    PASS = "pass"
    # Handed to a worker when its session launches. A running session keeps what it launched with; every session
    # launched after a write gets the new value.                                  product:session-lifetime
    SESSION = "session"
    # Read from the file when the daemon starts, held until it stops. Never an entry, never written through the API;
    # changing it is a file edit and a restart. The one lifetime an env variable may supply. product:boot-lifetime
    BOOT = "boot"
    # Written once, when the thing it belongs to is created; every later write is refused, across restarts too.
    IMMUTABLE = "immutable"                                                     # product:immutable-lifetime


def is_entry(lifetime: Lifetime) -> bool:
    # An entry lives in its own daemon's store, is written only through its own daemon's API, and is seeded from
    # the file (lifetimes.md §Seed). Neither daemon's API reads or writes the other's entries.
    return lifetime is not Lifetime.BOOT
```

A key's path below is as its daemon's contract document spells it (`contracts/config/hub.json`,
`contracts/config/host.json`). Deprecated keys read only to be refused, graduated, or honored are left out, but for the
two marked below; code paths are under `src/blizzard/`. A table key takes the lifetime of its children; a key whose
children differ is listed child by child.

## Data

### The one new setting

| Setting        | Daemon | Lives in                      | `lifetime` | Default | Replaces                                                       |
| -------------- | ------ | ----------------------------- | ---------- | ------- | -------------------------------------------------------------- |
| `tick_seconds` | runner | a host-level host store entry | `pass`     | `30.0`  | `BZ_RUNNER_TICK_SECONDS` (`runner/cli/runtime.py`), deprecated |

`tick_seconds` drives every runner's tick and the host tick. Every other key below is an existing file key; this epic
changes where it lives and when it is evaluated.

### Runner host, host-level keys

| Key                                                                                                                              | `lifetime`         | Held today                                                                                                                                                                                                                           |
| -------------------------------------------------------------------------------------------------------------------------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `host`, `port`, `public_url`, `trusted_proxies`                                                                                  | `boot`             | The listener and the federation identity registered with the hub                                                                                                                                                                     |
| `db_url`                                                                                                                         | `boot`             | The runner's store                                                                                                                                                                                                                   |
| `hub_url`, `host_token`, `session_secret_env`                                                                                    | `boot`             | Every hub client and the UI session signer, built at start                                                                                                                                                                           |
| `hub_tokens`                                                                                                                     | `boot`, deprecated | Each runner's hub token by name; read only by the seed that imports it ([runner-store.md](./runner-store.md) §Sealed tokens)                                                                                                         |
| `auth.hub_role_default`, `auth.superuser`, `auth.users`                                                                          | `boot`             | The runner's auth wiring                                                                                                                                                                                                             |
| `loop`                                                                                                                           | `boot`             | Whether timer threads start at all (`runner/cli/runtime.py`)                                                                                                                                                                         |
| `flags`                                                                                                                          | `boot`             | The flag registry, resolved at load                                                                                                                                                                                                  |
| `tools.env_passthrough`, `tools.path_prepend`                                                                                    | `session`          | The tool profile handed to each spawned process                                                                                                                                                                                      |
| `tracing.platform`, `tracing.platform_sample_ratio`                                                                              | `boot`             | The platform tracer provider, installed once                                                                                                                                                                                         |
| `tracing.harness_telemetry`, `tracing.worker_programs`, `tracing.worker_program_services`                                        | `session`          | The telemetry environment handed to a worker at launch                                                                                                                                                                               |
| `tracing.sweep_seconds`, `tracing.settle_seconds`, `tracing.max_lag_seconds`, `tracing.batch_limit`, `tracing.replay_max_window` | `pass`             | The lease trace sweep, built once per runner                                                                                                                                                                                         |
| `transcripts.ship`                                                                                                               | `pass`             | Whether the transcript drain runs                                                                                                                                                                                                    |
| `transcripts.record_max_bytes`, `transcripts.chunk_max_bytes`                                                                    | `pass`             | The transcript pump                                                                                                                                                                                                                  |
| `worker_stdout.retention_days`                                                                                                   | `pass`             | `LoopConfig`, read by the retention pass                                                                                                                                                                                             |
| `workspace[].id`                                                                                                                 | `immutable`        | Operator-chosen, host-scoped, no default. Reported at each host tick (`wire/host.py`), declared in every binding, recorded on every claim's route (`hub/domain/runners/route.py`)                                                    |
| `workspace[].root`, `workspace[].provider`, `workspace[].repo[]`                                                                 | `pass`             | Built into the workspace's environment provider at boot (`runner/composition.py::build_workspaces`, `runner/environments/factory.py`)                                                                                                |
| `workspace[].envs`, `workspace[].max_environments`                                                                               | `pass`             | The environment pool and its cap, built into the provider                                                                                                                                                                            |
| `workspace[].base_branch`                                                                                                        | `pass`             | Built into the provider                                                                                                                                                                                                              |
| `workspace[].prompt`, `workspace[].prompt_file`, `workspace[].prompt_package`                                                    | `session`          | The workspace prompt, resolved once at start                                                                                                                                                                                         |
| `subscription[].slug`, `subscription[].provider`                                                                                 | `immutable`        | The slug keys usage samples, usage-limit marks, every `runner[].subscription`, and the hub's declaration (`runner/composition.py::host_report`); the provider selects the sampler, the renewer, and the shape of their usage history |
| `subscription[].name`                                                                                                            | `pass`             | A label, reported at each host tick                                                                                                                                                                                                  |
| `subscription[].credentials_path`, `subscription[].usage_api_base`                                                               | `pass`             | Built into the subscription's sampler and credential renewer at boot (`runner/composition.py`). A worker reads `credentials_path` at launch, so for a running session it is `session`                                                |
| `subscription[].sample_interval_seconds`                                                                                         | `pass`             | The sampler's and renewer's cadence within the host tick                                                                                                                                                                             |
| `tick_seconds` (new)                                                                                                             | `pass`             | —                                                                                                                                                                                                                                    |

A moved workspace or subscription key is taken up by the host tick's rebuild ([passes.md](./passes.md) §The host tick);
the next `acquire` uses the rebuilt provider.

### Runner host, per-runner keys (`runner[]`)

| Key                                                                               | `lifetime`  | Held today                                                                                                                                     |
| --------------------------------------------------------------------------------- | ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`                                                                              | `immutable` | Minted by the hub at `runner add`; keys the runner's store rows and leases, and every worker's `BLIZZARD_RUNNER_URL` path (`runner/config.py`) |
| `name`                                                                            | `pass`      | `LoopConfig.runner_name`, declared at every registration (`runner/loop/context.py`); a label, keyed by nothing once tokens are sealed by id    |
| `subscription`                                                                    | `pass`      | `LoopConfig.subscription`, the slug of the declared subscription the runner spends                                                             |
| `transcripts_root`                                                                | `session`   | Where the runner reads its harness transcripts from                                                                                            |
| `projects[]`                                                                      | `pass`      | `LoopConfig.projects` and `bindings`; declared to the hub at the next registration                                                             |
| `agents`                                                                          | `pass`      | `LoopConfig.agents`                                                                                                                            |
| `gates`                                                                           | `pass`      | `LoopConfig.gates`                                                                                                                             |
| `queue.strict`                                                                    | `pass`      | `LoopConfig`, read by the claim step                                                                                                           |
| `throttle.budget.usd`, `throttle.budget.window_hours`                             | `pass`      | `LoopConfig.throttle`, read at each claim decision                                                                                             |
| `throttle.pace.lead_pct`, `throttle.pace.window`, `throttle.pace.taper[]`         | `pass`      | As above                                                                                                                                       |
| `throttle.schedule.days`, `throttle.schedule.hours`                               | `pass`      | As above                                                                                                                                       |
| `throttle.utilization.below_pct`, `throttle.utilization.windows`                  | `pass`      | As above                                                                                                                                       |
| `throttle.on_decline`, `throttle.on_usage_limit`                                  | `pass`      | As above                                                                                                                                       |
| `cost.chunk_cap_usd`, `cost.runner_ceiling_usd`, `cost.window_hours`              | `pass`      | Read at each spend check                                                                                                                       |
| `context.sample_interval_seconds`                                                 | `pass`      | `LoopConfig`, the context-sample cadence                                                                                                       |
| `context.warn_tokens`                                                             | `pass`      | Read at each sample                                                                                                                            |
| `models.aliases`, `models.supported`, `effort.aliases`                            | `session`   | The Claude Code adapter's resolution tables; `models.supported` is also advertised at the next registration                                    |
| `opencode.models.aliases`, `opencode.models.supported`, `opencode.effort.aliases` | `session`   | The OpenCode adapter's resolution tables, as above                                                                                             |
| `claude_code.enabled`, `opencode.enabled`                                         | `pass`      | Which harnesses the runner advertises                                                                                                          |
| `claude_code.binary`, `opencode.binary`                                           | `session`   | The binary a launch execs; re-probed for its version at the next health refresh                                                                |
| `opencode.auth_path`, `opencode.worker_config_path`, `harness.config_dir`         | `session`   | Paths handed to a worker at launch                                                                                                             |
| `harness.autonomy`                                                                | `session`   | The permission mode composed into a worker's settings                                                                                          |
| `worker.env_passthrough`, `worker.path_prepend`, `worker.shell`                   | `session`   | The worker environment built at launch                                                                                                         |
| `runner_prompt`, `runner_prompt_file`                                             | `session`   | Resolved by `LoopWiring.of` when the stepper is built                                                                                          |

What a lowered `agents`, a disabled harness, or a removed project does to running sessions is [passes.md](./passes.md)
§A runner's tick.

### Hub

Every hub value below is bound into a long-lived object in `hub/composition.py::build_services` today, or read from the
boot configuration per request. The `pass` keys move to the hub-wide record ([hub.md](./hub.md)).

| Key                                                                                                                       | `lifetime`         | Held today                                                                                                      |
| ------------------------------------------------------------------------------------------------------------------------- | ------------------ | --------------------------------------------------------------------------------------------------------------- |
| `host`, `port`, `public_url`, `trusted_proxies`, `db_url`                                                                 | `boot`             | The listener, the store, and the public identity                                                                |
| `auth.mode`, `auth.superuser`, `auth.oauth.provider[]`                                                                    | `boot`             | The auth wiring and OAuth clients                                                                               |
| `flags`                                                                                                                   | `boot`             | The flag registry                                                                                               |
| `follow_latest`                                                                                                           | `boot`             | Read at start                                                                                                   |
| `auth.login_throttle.enabled`, `.capacity`, `.refill_seconds`                                                             | `pass`             | `IpThrottle` (`hub/composition.py`)                                                                             |
| `action_pass_seconds`                                                                                                     | `pass`             | `ActionPassDriver`'s `PassBound` (`hub/composition.py`)                                                         |
| `annotation_interval_seconds`                                                                                             | `pass`             | The annotation sweep's interval and `AnnotationReconciler.full_pass_floor`                                      |
| `sweeps.<name>.mode`                                                                                                      | `boot`             | Whether `lifespan` starts the sweep's timer task (`hub/app.py`)                                                 |
| `close_forge_writes_enabled`                                                                                              | `pass`             | Consulted wherever a closer is handed out                                                                       |
| `produces_mode`, `route_token_mode`                                                                                       | `pass`             | Read where a step's produces and route token are judged                                                         |
| `transcripts.record_max_bytes`, `transcripts.chunk_budget_max_bytes`, `transcripts.runner_daily_rate_max_bytes`           | `pass`             | The transcript ingest service                                                                                   |
| `tracing.platform`, `tracing.platform_sample_ratio`                                                                       | `boot`             | The platform tracer provider                                                                                    |
| `tracing.sweep_seconds`, `tracing.settle_seconds`, `tracing.max_lag_seconds`, `tracing.batch_limit`                       | `pass`             | `TraceExportSweep` (`hub/domain/observability/tracing/sweep.py`)                                                |
| `tracing.replay_max_window`                                                                                               | `pass`             | `TraceReplay`                                                                                                   |
| `egress.directory`, `egress.format`, `egress.file_paths`, `egress.path_key_env`                                           | `boot`             | The egress writer and the process token it mints at start                                                       |
| `egress.datasets`, `egress.extractor_versions`, `egress.batch_limit`, `egress.max_rows_per_file`, `egress.min_free_bytes` | `pass`             | `EgressWriterSettings` (`hub/composition.py`)                                                                   |
| `egress.sweep_seconds`, `egress.settle_seconds`                                                                           | `pass`             | `EgressSweep` (`hub/domain/observability/egress/sweep.py`)                                                      |
| `egress.backfill_max_window`                                                                                              | `pass`             | The backfill command's bound                                                                                    |
| `work_source[]`                                                                                                           | `boot`, deprecated | Read at start only to refuse it and name its import; `replaced_by` the work-source records (`epic:live-config`) |

## Lifetimes for constants the sweep will promote

The lifetime each constant declares when `epic:config/sweep` promotes it
(`engineering:config/promoted-constant-lifetime`). This epic promotes none of them.

| Constant                                                                          | Daemon | Where                                                                                                | Value               | `lifetime` |
| --------------------------------------------------------------------------------- | ------ | ---------------------------------------------------------------------------------------------------- | ------------------- | ---------- |
| `HEARTBEAT_STALENESS_THRESHOLD`                                                   | runner | `runner/leases/activity.py`                                                                          | 1 h                 | `pass`     |
| `ELICITATION_STALENESS_THRESHOLD`                                                 | runner | `runner/leases/elicitation.py`                                                                       | 15 min              | `pass`     |
| `DEFAULT_RETRIES_MAX`                                                             | runner | `runner/loop/context.py`                                                                             | 2                   | `session`  |
| `DEFAULT_CHECK_TIMEOUT`                                                           | runner | `runner/lifecycle/judgement/check_runner.py`                                                         | 600 s               | `session`  |
| `SHUTDOWN_DRAIN_DEADLINE`                                                         | runner | `runner/lifecycle/shutdown_drain.py`                                                                 | 60 s                | `pass`     |
| overload backoff base, cap, limit                                                 | runner | `runner/leases/overload.py`                                                                          | 60 s, 900 s, 5      | `pass`     |
| `FRESH_SAMPLE_INTERVALS`, `FLOOR_UNCONFIRMED_GRACE`, `FLOOR_NO_RESET_FALLBACK`    | runner | `runner/throttle/levers.py`, `runner/throttle/floor.py`                                              | 3, 1 h, 30 min      | `pass`     |
| `DRAIN_LIMIT`, `HOST_LANE_DRAIN_LIMIT`, `RETENTION_FLOOR`                         | runner | `runner/lifecycle/drain.py`, `runner/lifecycle/host_lane_drain.py`, `runner/loop/retention_floor.py` | 100, 100, 1 h       | `pass`     |
| provisioning timeouts                                                             | runner | `runner/environments/internal/`                                                                      | 120, 60, 60, 300 s  | `session`  |
| hub HTTP timeout for the loop                                                     | runner | `runner/loop_wiring.py::_HTTP_TIMEOUT`                                                               | 30 s                | `pass`     |
| store retention windows                                                           | runner | `runner/store/internal/`                                                                             | 7 d, 8 d, 1 d, 1 d  | `pass`     |
| `EVENT_DERIVATION_INTERVAL_SECONDS`, `TENANT_TEARDOWN_INTERVAL_SECONDS`           | hub    | `hub/app.py`                                                                                         | 60 s, 30 s          | `pass`     |
| `CLOSE_DRAIN_INTERVAL_SECONDS`, `CLOSE_DRAIN_BACKOFF_BASE_SECONDS`                | hub    | `hub/app.py`, `hub/domain/work_items/closure.py`                                                     | 60 s, one setting   | `pass`     |
| flow driver `TICK_SECONDS`, `WAKE_FLOOR_SECONDS`                                  | hub    | `hub/delivery/flow_driver.py`                                                                        | 2 s, 1 s            | `pass`     |
| `STUCK_AFTER`, `DEFAULT_POLL_INTERVAL`, `DEFAULT_POLL_TIMEOUT`, `PARK_POLL_FLOOR` | hub    | `hub/domain/delivery/flows.py`                                                                       | 3, 30 s, 3 h, 2 min | `pass`     |
| `DEFAULT_BOUNCE_CAP`                                                              | hub    | `hub/domain/graph/model.py`                                                                          | 5                   | `session`  |
| forge attempt timeout and retry delays                                            | hub    | `hub/forge/internal/github_landing.py`                                                               | 30 s, (1 s, 3 s)    | `pass`     |
| session `IDLE_TTL`, `ABSOLUTE_MAX_AGE`                                            | hub    | `hub/auth/service.py`                                                                                | 24 h, 30 d          | `pass`     |

## Pinned

Constants with no lifetime. They stay in code and are never promoted (`product:declared-lifetime`):

- cryptographic sizes;
- wire-format limits (`wire/release.py`, `wire/analytics.py`);
- slug and name lengths;
- GitHub's own limits mirrored in `hub/forge/internal/github_landing.py`;
- the admitted harness version ranges (`runner/harness/claude_code/health.py`,
  `runner/harness/opencode/compatibility/probe.py`);
- the Anthropic credential renewal lead inside Claude Code's own refresh threshold;
- the usage-limit retention window above one week.
