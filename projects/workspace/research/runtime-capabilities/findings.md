# Runtime capability findings

Read-only reverse engineering of the Python Workspace runtime and three first-party providers. Evidence paths below are
relative to the development workspace's repository collection, not host paths. Source and test inspection establish
observed behavior; no live commands, provider mutations, or tests were run for this investigation.

The requirements live in `initiative:running-environments`. These findings are a migration inventory, not a frozen
command, configuration, result, or provider protocol specification. `decision:rough-parity`,
`decision:rust-replacement`, and `decision:requirements-first` distinguish capability continuity from preserving
incidental Python behavior.

## Charter outcome versus current enforcement

The Workspace charter promises a working environment from one safe-to-repeat command, including repositories/branches,
provisioned resources, and running services. Current context instead requires the operator to run structural init,
provision, then services; provisioning tracks no successful-run ledger, and the service startup implementation does not
prove prior provisioning. Do not describe a persisted readiness gate as existing behavior.

`product:running-environments/resource-provisioning/composed-environment-bringup` owns the target composition at
requirement level: structural reconciliation, then preparation, then workload launch, with no workload launch after
failed preparation. Preparation infrastructure explicitly required by a handler is a distinct, disclosed integration.
Readiness evidence, persistence, and invalidation remain open under
`decision:running-environments/service-orchestration/readiness-enforcement`.

Evidence:

- `winter-planning/projects/workspace/charter/mission.md`: one-command working environment success criterion.
- `winter-workspace/context/winter/environment-lifecycle.md`: provision-before-run workflow baseline.
- `winter-workspace/context/winter/cli/usage/provision.md`: structural init versus preparation, handler idempotency.
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/provision/provision_service.py`: pipeline, no readiness
  ledger.
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/service/handler.py`: workspace ensure, startup, optional
  health wait; no persisted provision check.

## Old surface to owning epic

| Existing surface or capability                                                                | Owning epic / integration                                                                                                                                         |
| --------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `winter env`, `--resolve`, scope bands, references, command-valued imports, masking           | `epic:running-environments/scope-environment`                                                                                                                     |
| Index-derived runtime ports and service namespace                                             | Scope environment consumes `epic:parallel-workspaces/environment-lifecycle` identity; nested allocation remains with `epic:parallel-workspaces/nested-workspaces` |
| `winter provision`, stages, named entries, apply/reset/destroy, resource-plus-data            | `epic:running-environments/resource-provisioning`                                                                                                                 |
| `winter clean`, declared artifact removal, preview, NDJSON                                    | `epic:running-environments/resource-provisioning`; not git-level `ws clean`                                                                                       |
| Provision `required_services` and static service catalog membership                           | Resource provisioning integration with service orchestration; diagnostics consume findings                                                                        |
| Resource teardown during structural environment destroy                                       | Resource provisioning requirement-level integration with environment lifecycle                                                                                    |
| Safe-to-repeat one-command working environment                                                | Resource provisioning composed-bringup requirement integrates lifecycle and service orchestration                                                                 |
| Capability binding, version compatibility, local provider override, common service operations | `epic:running-environments/service-orchestration`; external execution consumes `epic:agent-ready-operation/executable-extensions`                                 |
| Extension service contributions, describe ownership, catalog                                  | Service orchestration consumes extension declarations; each provider owns supported translation and continuity                                                    |
| `service up/down/start/stop/restart/status`, `--wait`, workspace infrastructure               | `epic:running-environments/service-orchestration`; exact aliases/vocabulary remain a project decision                                                             |
| `service logs`, tail/time filtering, follow, interruption                                     | `epic:running-environments/service-orchestration`, plus provider-specific sources and fairness                                                                    |
| Tmux sessions, layout, panes, health, capture, env-root convenience doors, cleanup            | `epic:running-environments/provider-tmux`                                                                                                                         |
| Compose files/projects, container health, published ports, volumes, backend logs              | `epic:running-environments/provider-docker`                                                                                                                       |
| Headless supervisor instances, update/restart, sockets, process state, orphan recovery        | `epic:running-environments/provider-process-compose`                                                                                                              |
| Provider prerequisite/health diagnostics                                                      | Each provider supplies truthful checks; `epic:agent-ready-operation/workspace-diagnostics` integrates operator reporting                                          |
| Legacy declaration conversion and release parity                                              | `epic:rust-adoption/workspace-migration` and `epic:rust-adoption/parity-validation`; not hidden compatibility fallbacks                                           |

## Provisioning and cleanup inventory

### Execution and targeting

- Workspace config and eligible installed-extension manifests contribute the same handler shape. Extension-eligible
  repositories include project repositories carrying a root manifest, not just standalone extension checkouts.
- Full apply runs dependency, resource, data. Within each stage the order is workspace, environment, worktree; project
  config precedes extension sources within each scope, then original declaration order.
- Each action is a string or list of shell commands, each separately run with `sh -c`. A list does not preserve shell
  state across commands. No script existence, executable-bit, path-escape, or sandbox check is implied.
- Worktree handlers iterate configured project repositories. Environment handlers can select a declared project cwd;
  missing selected worktrees are hard errors except during best-effort clean.
- Handler environment is inherited process state plus extension context and, for feature scopes, the identity/index/port
  trio. It does not use complete env-band resolution, unlike service dispatch.
- `--name` selects one scope-qualified entry. Names are unique within scope across stages in one manifest; duplicates
  across sources instead become a runtime ambiguous-selector error. Stage mismatch and missing selectors fail.
- Multiple environment literals/globs are deduplicated and deterministic. An unmatched glob succeeds without work; a
  nonexistent literal reaches the missing-environment hard error.

### Failure and partial results

- Commands stop on first failure within a cwd. Ordinary nonzero worktree results accumulate across worktrees; a launch
  `OSError` returns earlier completed runs and aborts the remaining cwd executions.
- Apply failure stops remaining handlers/stages. Destroy/reset failure stops that selected operation; composed reset
  never applies after its destroy leg fails. No automatic rollback occurs.
- Clean selects only declared clean handlers and is best-effort across handler and stage failures. An explicitly named
  handler without clean warns; an empty clean stage starts no prerequisite service.
- Missing-project cwd under clean is folded into a handler error. Whole-run precondition failures still abort clean;
  service-check exceptions produce a closing final event before propagating.
- Unsuccessful per-environment summaries allow subsequent environments; uncaught exceptions can still escape the outer
  environment loop. Requirements must preserve partial results at this boundary rather than reproduce the exception gap.
- NDJSON includes run/stage/command progress, per-handler cwd results, warnings, plans, and final status. Handler-level
  aggregate exit is reduced to success/failure even though individual cwd exits survive.

### Cleanup and service integration

- Missing destroy warns and skips. Reset prefers explicit reset, else destroy-plus-apply, else warns/reapplies.
- Artifact clean is not git cleaning, has no confirmation, and reaches ignored output. Workspace clean runs at the live
  shared workspace root and repeats per selected environment, not once per aggregate run.
- Preview runs neither handlers nor services, but does not prove real cwd availability. Missing destroy/degraded reset
  warnings differ between real and preview paths; clean's explicitly named unavailable action warns in both.
- Required services are unioned stage-wide. Running-but-unhealthy satisfies the check; absent/unparseable status counts
  down. Check remediation starts whole owning scopes and does not reverify afterward. Started services remain running.
- Only workspace/current-environment service references are allowed; bypass skips checks and runtime token validation.
- Environment destroy invokes data destroy then resource destroy before extension shutdown hooks and structural removal,
  with service checking disabled; dependency destroy is excluded. Non-strict teardown continues but retains provision
  failure overall; strict teardown preserves the environment. Hooks may hide their own failures from strict mode.
- Malformed workspace provision entries are parsed on demand, not universally at workspace load; malformed extension
  manifests can be logged/skipped, making a nominally successful run incomplete.

Evidence:

- `winter-workspace/tools/winter-cli/src/winter_cli/modules/provision/manifest.py`
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/provision/provision_service.py`
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/provision/execution_service.py`
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/provision/service_check_service.py`
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/provision/handler.py`
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/workspace/destroy_service.py`
- `winter-workspace/tools/winter-cli/tests/modules/provision/test_provision_service.py`
- `winter-workspace/tools/winter-cli/tests/modules/provision/test_provision_execution_service.py`
- `winter-workspace/tools/winter-cli/tests/modules/provision/test_provision_service_check.py`
- `winter-workspace/tools/winter-cli/tests/modules/provision/test_provision_dry_run.py`
- `winter-workspace/tools/winter-cli/tests/modules/provision/test_provision_command_handler.py`
- `winter-workspace/tools/winter-cli/tests/modules/workspace/test_destroy_provision_teardown.py`
- `winter-workspace/context/winter/cli/configuration/provision.md`
- `winter-workspace/context/winter/cli/usage/clean.md`

## Scope environment inventory

- Workspace scope uses index 0 and does not compute a feature port base. Feature scopes consume recorded identity and
  receive feature/shared bases plus namespace. Port allocation itself belongs to lifecycle, not this resolver.
- Effective config is overlaid before workspace < feature < named precedence. Scalar strings/integers/floats are
  accepted, not booleans/arrays. Legacy `[env.vars]` fails load; dormant named bands are allowed with doctor warning.
- References support names and nonnegative integer offsets. Fixpoint resolution diagnoses undefined, cyclic, blocked,
  restricted, and withheld entries. Workspace-origin reads use restricted values and original managed base values,
  excluding the feature port base, even when another band redeclares a managed name.
- Pure managed-variable redeclarations are accepted, including self-extension from the original base; command entries
  under managed names fail config load, while imported managed output is silently dropped.
- Commands support raw/dotenv/flat JSON, optional shell, and exports allow-list. `_`-prefixed slots/output keys are not
  exported. Multi-value declared keys are labels, not necessarily values. Python scalar stringification affects JSON
  booleans/null and is a compatibility decision, not an automatic Rust rule.
- Default inspection and down/status mask declared command keys without execution. Keys only imported by unexecuted
  multi-value commands cannot be masked in advance; referencing them fails even if explicit resolution would succeed.
- Output collisions use band/declaration position for deterministic final winners. A third reader can still retain an
  earlier literal value before a higher-position import arrives: the code documents this intermediate-read limitation.
- Commands run at workspace root, tokenize after interpolation by default, inherit process state overlaid with visible
  scope, close stdin, and have a fixed 30-second timeout. Interpolation before tokenization makes quoting significant.
- Shell mode refuses Winter-token interpolation of command-derived values, but ordinary shell `$NAME` expansion from the
  supplied environment remains possible. The refusal does not provide a shell sandbox.
- Timeout, launch, decoding, tokenization, nonzero exit, and parse errors name the declared unrendered command. Child
  stderr can disclose secrets. Closed stdin does not prevent `/dev/tty` prompts or exit-zero empty values.
- Core caches one computation per scope per fan-out, but convenience wrappers, tmux mapping preflight, pane shells, and
  process-compose launch/restart wrappers can run new resolutions. End-to-end lookup frequency is not once per command.
- Template `ValueError` degrades service computation to an empty map even on startup; command `RepoError` aborts. Base
  provider context and inherited environment still exist, so collapse is neither safe launch nor hermetic teardown.

Evidence:

- `winter-workspace/tools/winter-cli/src/winter_cli/config/workspace.py`
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/workspace/env_provisioner.py`
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/workspace/env_band_resolver_service.py`
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/workspace/internal/subprocess_command_entry_runner.py`
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/service/provider_invocation.py`
- `winter-workspace/tools/winter-cli/tests/modules/workspace/test_env_provisioner.py`
- `winter-workspace/tools/winter-cli/tests/modules/workspace/test_env_band_resolver_service.py`
- `winter-workspace/tools/winter-cli/tests/modules/service/test_service_fan_out_service.py`, especially template-error
  degradation beside command-failure propagation.
- `winter-workspace/context/winter/cli/configuration/command-env-entries.md`
- `winter-workspace/context/winter/cli/configuration/command-entry-collisions.md`
- `winter-workspace/context/winter/cli/configuration/ports-and-environments.md`

## Common service and capability inventory

- The only current capability slot is service. Explicit single/list bindings coexist with implicit sole/all activation;
  lists deduplicate, implicit-all sorts names, and legacy `orchestrate_services` remains supported. Removed top-level
  `service_orchestrator` and `session_prefix` config keys fail load.
- Declared versions absent from old providers are accepted. Explicit validation and implicit dispatch do not apply every
  entrypoint/version check equivalently. Read-only introspection is not proof of operational compatibility.
- Local flag/environment overrides resolve provider directories or installed names and collapse configured fan-out for
  one invocation. They do not change installed extension context used elsewhere.
- Service-v1 defines up/down/status/restart/logs/describe/catalog, base extension context vars, scope injection on
  up/down/status, and up-only aggregated service manifest and timeout. External runtime languages are unconstrained.
- Single-provider operation skips describe; multi-provider describe constructs ownership and detects duplicate claims.
  Catalog is separate membership for lint; malformed catalog providers are silently omitted by legacy collection.
- Up/down/status use registry scope rows; wildcard scopes exclude shared workspace unless explicit. Bare status includes
  all. Up/down dispatch concrete cells; multiple service filters must not widen into whole-scope mutation.
- Feature up attempts workspace first then requested targets, retaining workspace failure. Requested up stops on first
  failed cell; down continues after failed cells. Feature down does not stop workspace infrastructure.
- Restart validates all env tokens first; multi-provider validation adds ownership checks. Logs/restart forward original
  provider-owned patterns rather than universally concretizing environment globs.
- Status cells run concurrently and merge in enumeration order. First nonzero in that order wins, not numeric maximum.
  Valid partial documents survive; read rendering diagnoses parse errors, while internal collect discards them.
- Core readiness asks whether anything is unhealthy. Unknown health, empty status, no cells, or no parseable document
  can pass. Core wait excludes implicit workspace ensure. This is not a general running/prepared guarantee.
- Logs carry provider NDJSON then render plain attributable output. Common backstops select services, apply inclusive
  time filtering where timestamps exist, and tail a global non-follow ring; follow does not re-tail backend backlog.
- Multi-provider follow merges observed line order, not timestamp order, with bounded queueing and cleanup of follower
  process groups on interruption. A provider exit does not stop peers. Non-follow dispatch is sequential.
- Up/restart strip launch trace context; provision and observation preserve relevant invocation context. Telemetry must
  not capture secret-bearing values or command output.
- Aggregated startup service definitions have limited fields and globally unique names; providers may ignore the file.
  Only up receives it. Its temporary file is created without automatic deletion in the collector, and later actions do
  not uniformly rediscover startup-only contributions.

Evidence:

- `winter-workspace/tools/winter-cli/src/winter_cli/modules/capability/capability_registry_service.py`
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/capability/version_compat.py`
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/capability/specs/service-v1.toml`
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/service/orchestrator_resolver.py`
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/service/service_dispatch_service.py`
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/service/service_status_matrix_service.py`
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/service/service_status_service.py`
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/service/service_readiness_service.py`
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/service/service_logs_service.py`
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/service/service_manifest_collector.py`
- `winter-workspace/tools/winter-cli/tests/modules/capability/test_capability_registry_service.py`
- `winter-workspace/tools/winter-cli/tests/modules/service/test_service_dispatch_service.py`
- `winter-workspace/tools/winter-cli/tests/modules/service/test_service_readiness_service.py`
- `winter-workspace/tools/winter-cli/tests/modules/service/test_follow_concurrency.py`
- `winter-workspace/context/winter/cli/contracts/service-orchestrator.md`

## Three independent provider tracks

### Tmux

The provider uses a dedicated shared `winter` server, sanitized startup environment, login pane shells, prefixed scope
sessions, config-relative layout hooks, scope-relative cwd, and shell-evaluated machine credentials. Interactive
empty-command services and attach are real capabilities. Whole-up of an existing session is a no-op; partial up launches
only matched stopped panes, skipping missing panes. Selected mappings preflight before mutation. Failed hooks can retain
multi-window partial sessions; missing declared pane targets tear down newly created sessions.

Dependencies use stable local ordering plus provider-agnostic status polling, with per-dependency timeout from the
injected timeout variable. Partial startup does not start dependency closure automatically. Startup retry is opt-in
process-exit monitoring, not health. URL, cmd, log-regex, and child-uptime probes are implemented. File capture persists
with rotation/retention; pane mode preserves TTY but lacks timestamps; memory mode is accepted but emits no logs.

Normal down recursively collects pane descendants, TERM/recollects/KILLs them, and then kills the session; signal errors
can merely warn. Destroy suppresses errors. With unreadable config, its fallback kills every shared-server session whose
name ends in the environment suffix, without validating workspace ownership. This is unsafe behavior to replace, not
rough parity to preserve. Process inspection relies on `pgrep` and `ps` flags whose OS parity needs verification.

Evidence:

- `winter-service-tmux/src/service_orchestrator/modules/orchestrate/orchestrator_service.py`
- `winter-service-tmux/src/service_orchestrator/modules/orchestrate/tmux_server.py`
- `winter-service-tmux/src/service_orchestrator/modules/orchestrate/status_report.py`
- `winter-service-tmux/src/service_orchestrator/modules/orchestrate/internal/pgrep_process_reaper.py`
- `winter-service-tmux/src/service_orchestrator/env_cli.py`, suffix fallback and delegated convenience doors.
- `winter-service-tmux/src/service_manifest/modules/manifest/ext_reader.py`, target-required contributions and skips.
- `winter-service-tmux/tests/modules/orchestrate/test_orchestrator_service.py`, preflight, salvage, targeted lifecycle,
  health, retry, and dependency cases.
- `winter-service-tmux/tests/modules/orchestrate/test_tmux_server.py`
- `winter-service-tmux/tests/modules/orchestrate/internal/test_pgrep_process_reaper.py`
- `winter-service-tmux/context/service-rules.md`

### Docker Compose

The provider partitions scope-pure feature/workspace compose files and names projects by prefix and scope. Whole up uses
argless compose startup, so the file itself is the scope boundary. Feature substitution ports are base plus project
service declaration position; reordering changes them. Workspace ports use shared variables. Status reports published
host ports and only the first container per service as representative.

Up always runs a raw-state readiness gate: running without a healthcheck or running/healthy passes; starting and exited
do not. Partial up delegates dependency side effects to compose and gates only returned selected containers. Timeout
currently uses provider-specific environment settings rather than the core timeout. Partial down stops/removes selected
containers; whole down removes the project but retains named volumes. It can clean a configured project even when the
current service list is empty.

Restart/logs do not enumerate wildcard environment patterns; tests expect refusal for restart. Follow drains matched
services sequentially, so one endless first stream starves later services. Status does not consistently retain failed
compose invocation status. The destroy shell hook still requires removed `compose_file` plus explicit `project_prefix`,
so current two-file/default-prefix setups can skip cleanup and report success. Aggregated startup service definitions
are not consumed by this provider.

Evidence:

- `winter-service-docker/src/docker_orchestrator/manifest.py`
- `winter-service-docker/src/docker_orchestrator/lifecycle.py`
- `winter-service-docker/src/docker_orchestrator/status.py`
- `winter-service-docker/src/docker_orchestrator/cli.py`, provider-specific timeout inputs.
- `winter-service-docker/src/docker_orchestrator/restart.py`
- `winter-service-docker/src/docker_orchestrator/logs.py`
- `winter-service-docker/hooks/destroy-worktree.sh`
- `winter-service-docker/tests/test_two_file_scope.py`
- `winter-service-docker/tests/test_lifecycle.py`
- `winter-service-docker/tests/test_restart.py`, wildcard-only scope refusal.
- `winter-service-docker/tests/test_status.py`
- `winter-service-docker/context/provider-contract.md`

### Process-compose

One daemon per scope has a lock, socket, generated config, daemon log, state, and service logs. Long sockets use a
private runtime fallback derived from the scope path. Missing-root operations avoid recreating it; down/restart may
reach a daemon at its fallback socket after root deletion, without the missing root's lock.

Fresh up launches disabled definitions then enables dependency waves to work around supervisor first-update behavior.
Repeated up updates definitions and preserves unchanged pids. Partial up includes transitive same-scope dependencies.
Partial down stops selected processes; whole down removes control/state. Stale recovery uses recorded session identity
and refuses when the daemon pid is still alive but unresponsive. Linux `/proc` session scans clean orphans; no recorded
birth fingerprint protects against identity reuse. Recovery behavior is not automatically portable.

Actual process launches source resolved Winter values; generated definitions avoid storing these command values, but
health targets are interpolated during rendering and can contain resolved material. Probe environment is not equivalent
to tmux's source/mapping snapshot. URL/cmd probes and restart policies are supported; interactive commands, log/uptime
probes, and cross-scope dependencies are refused. Target/log pane fields are ignored. Startup contributions without cmd
are skipped with notes; name collisions fail.

Up waits on declared health and fails early for terminal failure among probed selected services. Native restart
truncates logs. File following handles truncation and late creation, but the CLI loops scopes sequentially, so fair
multi-scope follow needs more than the existing per-service loop. Wildcard scopes are refused or lack concrete scope
enumeration for restart/logs. Doctor declares a 1.122.0 minimum; adapter calls are not independently deadline-bounded.

Evidence:

- `winter-service-process-compose/src/process_compose_orchestrator/cli.py`
- `winter-service-process-compose/src/process_compose_orchestrator/modules/instance/lifecycle_service.py`
- `winter-service-process-compose/src/process_compose_orchestrator/modules/instance/config_renderer.py`
- `winter-service-process-compose/src/process_compose_orchestrator/modules/instance/socket_layout.py`
- `winter-service-process-compose/src/process_compose_orchestrator/modules/instance/internal/proc_session_reaper.py`
- `winter-service-process-compose/src/process_compose_orchestrator/modules/instance/internal/cli_process_compose_client.py`
- `winter-service-process-compose/tests/test_lifecycle.py`, missing-root/fallback recovery and stale/live refusal.
- `winter-service-process-compose/tests/test_renderer_status_layout.py`
- `winter-service-process-compose/tests/test_manifest_reader.py`
- `winter-service-process-compose/tests/test_logs.py`
- `winter-service-process-compose/tests/test_live_adapters.py`
- `winter-service-process-compose/context/provider.md`

## Observed inconsistencies requiring conscious disposition

| Observation                                                                                                             | Consequence for requirements                                                |
| ----------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| Core service usage says tmux health is future work; provider source/tests implement four probe types                    | Preserve actual capability inventory, not the outdated warning              |
| Service status prose sometimes says worst/highest exit; matrix returns first nonzero in enumeration order               | Choose aggregate policy explicitly; retain individual failures              |
| Contract prose says env collapse removes service prefix; separately built base context retains it                       | Keep independently known identity; do not claim complete hermeticity        |
| Template error degrades startup env while command failure aborts                                                        | Refuse incomplete required launch environment consistently                  |
| No parseable status can pass core wait                                                                                  | No missing-status readiness success by accident                             |
| Provision schema docs imply universal load validation; tests preserve deferred workspace parsing and skipped extensions | Decide validation boundary, report incomplete contributions                 |
| Shared clean is repeated per feature target and has no confirmation                                                     | Choose shared reach, deduplication, and authorization deliberately          |
| Preview omits some destroy/reset warnings and does not prove cwd existence                                              | Show unavailable actions and validation limits, not false safety proof      |
| Generic cross-env selection exceeds Docker/process-compose restart/log support                                          | Common conformance includes actual selection capability or explicit refusal |
| Generic concurrent follow does not fix Docker per-service or process-compose per-scope sequential follow                | Require fairness within providers as well as across them                    |
| Docker destroy hook reads obsolete schema and ignores injected default prefix                                           | Native teardown must share ordinary lifecycle interpretation                |
| Tmux fallback suffix kill crosses workspace boundaries                                                                  | Require verified ownership or refusal after configuration loss              |
| Process-compose docs claim generated config has no resolved values, but probes interpolate                              | Define actual persistence exposure, lifetime, and access rules              |
| `ext verify` can pass recognition/refusal and argv echo without safe lifecycle behavior                                 | Require behavioral conformance, not self-echo as runtime proof              |
| Startup contributions arrive only on up and differ in provider support                                                  | Own declaration continuity and explicit capability gaps                     |

## Native workflow scope and retained prerequisites

A Rust core alone leaves Python in every first-party provider. All three shims execute Python modules, their package
metadata requires Python >=3.13, tmux uses Python for capture and relative symlink creation, and doctor/destroy helpers
retain separate shell logic. The target promise includes replacing those maintained first-party workflow obligations,
not just the top-level CLI. It does not require translating arbitrary user hooks, layout scripts, service commands,
compose files, or value-producing programs into Rust.

Retained prerequisites include tmux and supported shells/process facilities, Docker daemon plus Compose v2, and the
supported process-compose binary. Linux `/proc`, Unix endpoint lengths, advisory locking, inherited environments, login
profiles, and backend version quirks need explicit supported-system treatment rather than invented portability.
Execution placement of native built-ins remains open; external providers remain separate programs with arbitrary
implementation languages and the same Winter-owned semantic obligations.

Evidence:

- `winter-service-tmux/workflow/orchestrate`, `winter-service-tmux/workflow/up`
- `winter-service-tmux/hooks/init-worktree.sh`, `winter-service-tmux/hooks/destroy-worktree.sh`
- `winter-service-tmux/src/service_orchestrator/logwriter.py`
- `winter-service-docker/workflow/service`, `winter-service-docker/workflow/doctor.sh`
- `winter-service-docker/hooks/destroy-worktree.sh`
- `winter-service-process-compose/workflow/service`, `winter-service-process-compose/workflow/doctor.sh`
- `winter-service-process-compose/hooks/destroy-worktree.sh`
- `winter-service-tmux/pyproject.toml`, `winter-service-docker/pyproject.toml`,
  `winter-service-process-compose/pyproject.toml`

## Dependency shape and independent delivery

Scope environment consumes lifecycle registry identity; it does not create it. Resource preparation consumes scope
values and executable-extension support but can execute ordered stages without complete service orchestration. Only
required-service checks and composed working-environment bring-up depend at requirement level on service control and
observation. Service orchestration consumes scope values and executable extensions, not provisioning. Each provider
depends on service orchestration and is independently usable/deliverable without either sibling provider.

This avoids a registry -> provision -> service -> executable-extension -> scope-value -> registry epic cycle. Later
lifecycle integrations use the capabilities once available rather than making structural identity wait for the complete
runtime capabilities. Exact command vocabulary, readiness evidence, schemas, contracts, execution placement, and
implementation choices are intentionally not selected by these research findings.
