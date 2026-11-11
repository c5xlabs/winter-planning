# Operator capability evidence

Read-only inspection of the current Workspace source, tests, context, and separately versioned plugin API informs
`initiative:agent-ready-operation`. Tests below were read, not executed; existing behavior is evidence, not automatic
approval of its command spelling, output format, Python runtime, or inconsistent edge cases.

All evidence paths are repository-relative and begin with the owning repository name. Command names in this record
describe the legacy CLI, not the replacement vocabulary selected by `decision:command-vocabulary`.

## Legacy command to capability map

| Existing surface                                                             | Requirements owner                                                               | Boundary                                                                                             |
| ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| `winter graph`, `--json`                                                     | `epic:agent-ready-operation/dependency-graph`                                    | Declared extension-module adjacency, not all context links.                                          |
| `winter doctor`, `--json`                                                    | `epic:agent-ready-operation/workspace-diagnostics`                               | Health evidence and optional trusted checks, not automatic repair.                                   |
| `winter lint`, names/globs, `--all`, `--changed`, `--show-ignored`, `--json` | `epic:agent-ready-operation/context-quality`                                     | Content scope, check catalog, audited exemptions.                                                    |
| `winter dashboard`, overview/layout/detail/keymap                            | `epic:agent-ready-operation/workspace-dashboard`                                 | Shared observation meaning, navigation, and local failure history.                                   |
| Dashboard agent matrix and its reconcile action                              | `epic:agent-ready-operation/workspace-dashboard`                                 | Consumes agent resolution, harness projection, and bootstrap; does not own their mutation semantics. |
| `winter space <kind>`                                                        | `epic:agent-ready-operation/artifact-space`                                      | Pure destination resolution; not declared cleanup.                                                   |
| `winter ws status --json` layout/status evidence                             | `epic:parallel-workspaces/coordinated-git` with dashboard integration            | Versioned status observation is also a headless view of layout resolution.                           |
| `winter service logs`                                                        | `epic:running-environments/service-orchestration`                                | Provider streams, filters, follow/interruption; not the dashboard error log.                         |
| Root verbose/log-level and tracing                                           | Project engineering constraints and `epic:parallel-workspaces/component-runtime` | Cross-cutting output/observability, not a separate operator capability epic.                         |

## Doctor evidence

`winter-workspace/tools/winter-cli/src/winter_cli/modules/doctor/doctor_service.py` aggregates named observations and
reports failure when any probe fails; warnings alone allow success. Human output groups by source, while
`modules/doctor/doctor_reporter.py` emits started/result/finished events for machine consumption.

The source inventory includes native tool availability, Python runtime, configuration parseability, repository and
worktree presence/branches, symlink health, and the committed agent-entry shim in
`winter-workspace/tools/winter-cli/src/winter_cli/modules/doctor/core_probe_service.py`. Additional evidence:

- `winter-workspace/tools/winter-cli/src/winter_cli/modules/doctor/port_probe_service.py`: capacity and registry drift.
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/doctor/env_bands_probe_service.py`: inert env bands warn.
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/doctor/skill_probe_service.py`: skill projection
  discoverability/currentness; workspace probes run even with extension adoption disabled.
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/doctor/agent_probe_service.py`: stale/missing/orphaned agent
  copies, unresolved tiers, duplicate names, and override targets.
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/doctor/capability_probe_service.py`: invalid bindings,
  missing entry points, unsupported provider versions, implicit providers, and no-provider warnings.
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/provision/manifest_probe_service.py`: malformed provision
  declarations are individual findings rather than a reason to abandon every other check.

Workspace and extension checks are executable contributions, not sandboxed reads. Entry points are resolved within their
declaring directory, including symlinks, checked for existence/executability, and launched from workspace cwd; extension
invocations carry explicit extension/config/service context. Output parsing keeps valid observations, turns bad lines
into warnings, and adds an execution failure for nonzero exit. See
`winter-workspace/tools/winter-cli/src/winter_cli/modules/doctor/workspace_probe_service.py`,
`winter-workspace/tools/winter-cli/src/winter_cli/modules/doctor/extension_probe_service.py`, and
`winter-workspace/tools/winter-cli/src/winter_cli/modules/doctor/probe_parser.py`.

Documentation: `winter-workspace/context/winter/cli/usage/doctor.md` and
`winter-workspace/context/winter/cli/configuration/doctor.md`. Test evidence lives in
`winter-workspace/tools/winter-cli/tests/modules/doctor/`, particularly `test_doctor_service.py`,
`test_extension_probe_service.py`, `test_workspace_probe_service.py`, `test_port_probe_service.py`,
`test_skill_probe_service.py`, and `test_agent_probe_service.py`.

The Python version check describes the old runtime, not a retained prerequisite. Current diagnostic construction also
requires already-loaded configuration through `winter-workspace/tools/winter-cli/src/winter_cli/container.py`, so an
advertised config-parse probe is not evidence that malformed startup config reaches a normal probe run. Degraded startup
and finite contribution limits remain explicit decisions in the diagnostic epic.

## Quality scope, catalog, and suppression evidence

`winter-workspace/tools/winter-cli/src/winter_cli/modules/lint/scope_resolver.py` selects current env/all envs, explicit
project source checkout/env, or deduplicated sorted names/globs. Repo/env ambiguity and unknown literals fail, while an
unmatched glob no-ops. Standalone-only names are excluded. Changed scope unions dirty/staged/untracked and unpushed
files, falls back from upstream to the main remote reference, and parses NUL-delimited paths and rename destinations.

`winter-workspace/tools/winter-cli/src/winter_cli/modules/lint/handler.py` skips programs for an empty changed set.
`modules/lint/lint_service.py` collects core, workspace, and extension findings before one exemption filter.
`modules/lint/finding_parser.py` accepts location fields and the legacy name alias, retains valid output on unsuccessful
exit, and surfaces malformed output as warning findings. Full paths for those latter modules are beneath
`winter-workspace/tools/winter-cli/src/winter_cli/`.

Built-in catalog evidence:

- `winter-workspace/tools/winter-lint/extractability.py` and `winter-workspace/tools/winter-lint/README.md`: declared
  dependency direction, unknown modules, dependency cycles, cross-module imports, illustrative-block exemption, fenced
  code exclusion, graph query reuse, and checkout-local dependency declarations.
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/lint/core_lint_service.py` and
  `winter-workspace/tools/winter-cli/src/winter_cli/modules/lint/markdown_size.py`: injected/reference effective-byte
  limits, transitive eager imports, formatter-neutral whitespace/rule normalization, and raw/effective size evidence.
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/lint/internal/gitignore_repository.py`: exclude only
  untracked ignored candidates during file-size discovery; preserve candidates on uncertain lookup and retain tracked
  files matching ignore rules. Individually selected files are not removed by directory discovery filtering.
- `winter-workspace/tools/winter-cli/src/winter_cli/modules/lint/required_services_check.py`: declaration references
  checked against merged provider catalogs without needing a running provisioned environment.

Workspace and extension programs receive selected file/directory paths and a graph-query entry point; they must honor
that scope themselves. The dispatcher does not sandbox access or make a no-write guarantee. Relevant sources are
`winter-workspace/tools/winter-cli/src/winter_cli/modules/lint/workspace_lint_service.py`,
`winter-workspace/tools/winter-cli/src/winter_cli/modules/lint/extension_lint_service.py`, and
`winter-workspace/tools/winter-cli/src/winter_cli/modules/lint/scope_env.py`.

`winter-workspace/tools/winter-cli/src/winter_cli/modules/lint/ignore_service.py` applies repository and workspace rules
by union, with segment-aware relative matching, path-only or check-specific suppression, no suppression of fileless
failures, and post-check audit. Rules naming absent content warn; live rules outside a narrow scope remain quiet.
Unusable declarations suppress nothing. Symlinked directories are not recursively followed in the audit walk.
`winter-workspace/tools/winter-cli/src/winter_cli/modules/lint/lint_reporter.py` always counts suppressed findings and
prints their details only on request, grouped by the matching rule.

Documentation: `winter-workspace/context/winter/cli/usage/lint.md` and
`winter-workspace/context/winter/cli/configuration/lint.md`. Regression evidence includes
`winter-workspace/tools/winter-cli/tests/modules/lint/test_scope_resolver.py`, `test_lint_handler.py`,
`test_finding_parser.py`, `test_ignore_service.py`, `test_core_lint_service.py`, `test_required_services_check.py`,
`test_lint_reporter.py`, and `internal/test_gitignore_repository.py` in that same test directory, plus
`winter-workspace/tools/winter-lint/test_extractability.py`.

### Inconsistencies and source-precedence choices

- Docs describe contributors as scripts; `lint_service.py` counts outcomes, including separate core checks but grouped
  workspace/extension contributions even when an owner declares several scripts. The accounting decision is open.
- `core_lint_service.py` returns before all three core checks when the bundled extractability script is absent. The new
  catalog requirement rejects that silent coupling rather than preserving it.
- The no-match branch in `handler.py` emits ordinary text even with machine mode selected. The replacement retains the
  no-op outcome, not the mixed-output bug.
- Ignore docs say unusable declarations never fail, but `ignore_service.py` emits failures for unreadable manifests or
  workspace config, and `test_ignore_service.py::test_unreadable_manifest_is_reported_rather_than_raised_or_logged`
  asserts a failure. Severity is a decision, not a fact resolved by choosing whichever sentence is convenient.
- Git discovery errors can become empty path lists in the legacy changed resolver. The new requirements distinguish
  unavailable changed-set evidence from a genuinely empty set.
- Ignore rules and extractability declarations can come from the checked worktree while extension check programs come
  from source checkouts. The source-precedence decision explicitly owns this split. Version-bound framework help belongs
  to `epic:agent-ready-operation/context-navigation`; it must not silently substitute for working project content.

## Graph evidence and limits

`winter-workspace/tools/winter-cli/src/winter_cli/modules/graph/graph_service.py` reads extension-eligible manifests and
collects declared dependency lists. It does not traverse arbitrary documentation links, execute modules, or enforce
cycles; extractability applies rules later. Broken manifests are logged and skipped, yielding a potentially incomplete
successful adjacency map. `winter-workspace/tools/winter-cli/src/winter_cli/modules/graph/graph_reporter.py` orders
modules deterministically for both readable and machine use.

`winter-workspace/context/winter/cli/usage/graph.md` documents the query and reuse by external checks. Tests are
`winter-workspace/tools/winter-cli/tests/modules/graph/test_graph_service.py` and
`winter-workspace/tools/winter-cli/tests/modules/graph/test_graph_reporter.py`. Completeness handling is an explicit
replacement decision; context reachability is proposed expansion, not inferred parity.

## Dashboard observations, navigation, and failure evidence

`winter-workspace/tools/winter-cli/src/winter_cli/modules/workspace/workspace_snapshot_service.py` and
`winter-workspace/tools/winter-cli/src/winter_cli/modules/workspace/env_status_service.py` provide shared CLI/TUI
observation meaning, with strict versus tolerant failure policies and different detail costs. The dashboard includes
singletons and standalones, worktrees, main-checkout indicators, tracking divergence and unborn upstream markers,
multi-remote labels, layouts, scoped selection, details, and service badges. The layout policy is also observable in
non-interactive workspace status through `winter-workspace/tools/winter-cli/src/winter_cli/config/models.py`.

`winter-workspace/tools/winter-cli/src/winter_cli/modules/workspace/dashboard_snapshot_service.py` reloads config and
rebuilds snapshot collaborators per overview refresh; malformed later config falls back to last-good inputs. Detail
views and some action collaborators retain launch-time instances. Keymaps and contributions are static for the session.
This is evidence of partial live reload, not a coherent all-settings freshness guarantee.

UI evidence is beneath `winter-workspace/tools/winter-cli/src/winter_cli/modules/tui/`:

- `screens/workspace/screen.py`, `screens/workspace/feature_worktrees.py`, and `screens/workspace/standalone_repos.py`:
  overview, four layout choices, focus and selection routing, manual/30-second refresh, and configured action bindings.
- `screens/worktree_detail/screen.py`, `screens/standalone_detail/screen.py`, and `widgets/repo_detail_view.py`:
  selected-repository detail, dirty paths, scrollable topology, upstream conditions, and isolated contributed panels.
- `keybindings/`: stable actions, modifiers/leader/sequences, timed prefixes, invalid-spec fallback, overlapping-area
  conflict reporting, and disjoint-area action sharing.
- `app.py` and `screens/plugin_action_mixin.py`: unconditional quit escape, worker cancellation, and guards against late
  callbacks during teardown.
- `screens/agent_matrix/screen.py` and `ws_init_runner.py`: current-config agent inspection and mutating bare workspace
  reconciliation; session-wide single-flight, output capture, non-interactive input, disabled Git HTTPS prompts, warning
  without cancellation after 60 seconds, outcome-once handling across reopen, and stale-load exclusion. Direct terminal
  SSH prompts are an acknowledged remaining gap.
- `error_log.py` and `screens/error_log/screen.py`: session-local structured error history, clear, and notification
  deduplication; legacy retention is unbounded and not persistent.

Loader/decorator/panel failures have isolation, but action handlers are directly called in worker paths without uniform
local log capture; some tolerated snapshot failures only emit warnings. Replacement requirements make captured failures
inspectable rather than asserting every legacy path already does so. The refresh/mutation concurrency decisions also
avoid treating Python thread/GIL behavior or a UI-only lock as authoritative safety across processes.

Documentation: `winter-workspace/context/winter/cli/usage/dashboard.md` and
`winter-workspace/context/winter/cli/configuration/tui.md`. Tests in
`winter-workspace/tools/winter-cli/tests/modules/tui/` include `test_feature_worktrees_grid.py`,
`test_worktree_detail_focus.py`, `test_detail_panels.py`, `test_standalone_action.py`,
`test_dashboard_config_reload.py`, `test_dashboard_quit.py`, `test_dashboard_error_capture.py`,
`test_agent_matrix_screen.py`, `test_ws_init_runner.py`, `test_error_log.py`, and `keybindings/`. Also see
`winter-workspace/tools/winter-cli/tests/modules/workspace/test_dashboard_snapshot_service.py`.

## Full contribution inventory, not just badges

`winter-workspace/tools/winter-cli/src/winter_cli/plugins/loader.py` discovers workspace-local, user-global, then
standalone-extension plugins, with first-name-wins precedence and load-failure isolation. Project-repo modules in
doctor/lint/graph are deliberately not TUI discovery sources; see
`winter-workspace/tools/winter-cli/src/winter_cli/container.py`. The legacy loader imports arbitrary Python in-process
through `winter-workspace/tools/winter-cli/src/winter_cli/plugins/internal/importlib_plugin_loader.py`.

The five contribution kinds and replacement implications are:

| Legacy contribution              | Observable value                                                                         | Replacement scope question                                                                      |
| -------------------------------- | ---------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| Environment/worktree decorators  | Refresh-time owned status badges and extension-private values.                           | Data observations must not claim native status or another owner's identity.                     |
| Detail panels                    | Named extra tabs returning markup or arbitrary Rich renderables.                         | Supported data-described detail content is not automatic compatibility with renderable objects. |
| Keybound action callbacks        | Workspace/env/worktree/standalone and multi-area actions, siblings, terminal suspension. | External actions need deliberate scope and terminal handoff, not serialized callables.          |
| Click command objects            | Extra command entry points on the running tool.                                          | Decide external command integration, separately launched tools, or explicit retirement.         |
| Arbitrary Textual screen classes | Entire custom interactive experiences.                                                   | Decide supported described views, external experiences, or explicit retirement.                 |

`winter-plugin-api/src/winter_plugin_api/seam.py` and `winter-plugin-api/src/winter_plugin_api/views.py` define the
separately versioned typed surface; `winter-plugin-api/README.md` describes the hand-maintained copy relationship with
`winter-workspace/tools/winter-cli/src/winter_cli/plugins/types.py`. Read-only views constrain typing, not runtime
authority: actual objects are passed structurally, extension maps are mutable, and Python code is unrestricted.
`winter-plugin-api/tests/test_contract.py` exercises self-conformance but does not prove cross-repository seam parity;
`winter-workspace/tools/winter-cli/tests/plugins/test_loader.py` covers loading, precedence, and failure isolation.

The replacement decision rules out importing Python, embedding Python, and a Python compatibility host. External
programs remain free to choose their language. Commands and arbitrary screens therefore require explicit scope/migration
decisions rather than a claim that replacing badge values preserves the complete plugin experience.

## Artifact destination evidence

`winter-workspace/tools/winter-cli/src/winter_cli/modules/space/command.py` validates a single ASCII kind and prints the
absolute destination; `winter-workspace/tools/winter-cli/src/winter_cli/config/models.py` resolves default, relative,
home-relative, and absolute choices. The resolver neither creates the directory nor cleans or claims its contents.
Trusted configured absolute roots and kind overrides intentionally escape `.winter`; source resolution is lexical/home
expansion rather than a requirement that the destination exists or can be canonicalized.

`winter-workspace/context/winter/cli/usage/space.md` and `winter-workspace/context/winter/cli/configuration/space.md`
document caller materialization and optional excludes.
`winter-workspace/tools/winter-cli/tests/config/test_workspace_config_service.py` covers root/kind combinations and
unknown kinds; `winter-workspace/tools/winter-cli/tests/test_cli.py` includes process-level valid/invalid kind exits.
Declared clean actions belong to `epic:running-environments/resource-provisioning`, not this pure query.

## Reporting, logging, and handoffs to other owners

The legacy CLI has no universal output envelope: graph is an adjacency document; doctor/lint and operation reporters use
event streams; workspace status is a versioned snapshot; service status is a structured document; space is a plain path;
service logs are rendered lines. This informs capability and compatibility decisions, not a requirement to freeze every
old field before `decision:command-vocabulary` is made.

Semantic reporter evidence includes
`winter-workspace/tools/winter-cli/src/winter_cli/modules/workspace/reporter_factory.py`,
`winter-workspace/tools/winter-cli/src/winter_cli/modules/workspace/init_reporter.py`, and the reporter files in
`winter-workspace/tools/winter-cli/src/winter_cli/modules/{doctor,lint,graph,service,agents,capability,provision,ext}/`.
`winter-workspace/tools/winter-cli/src/winter_cli/core/internal/click_cli_output_service.py` aligns styled human table
cells independently of color codes. Output-channel and compatibility requirements remain with component/runtime owners.

Service stream evidence is in `winter-workspace/tools/winter-cli/src/winter_cli/modules/service/service_logs_service.py`
and `winter-workspace/tools/winter-cli/src/winter_cli/modules/service/log_stream_processor.py`: selection backstops,
missing timestamp warnings, global non-follow tail, observed-order concurrent follow, separate stderr, process-group
cleanup, and interrupt exit 130. Tests are
`winter-workspace/tools/winter-cli/tests/modules/service/test_service_logs_service.py` and
`winter-workspace/tools/winter-cli/tests/modules/service/test_log_stream_processor.py`; service orchestration owns
replacement acceptance, not this dashboard-local error-history epic.

Global diagnostics/tracing evidence is `winter-workspace/tools/winter-cli/src/winter_cli/cli.py`,
`winter-workspace/tools/winter-cli/src/winter_cli/core/tracing.py`,
`winter-workspace/tools/winter-cli/src/winter_cli/core/internal/otel_command_tracer.py`, and
`winter-workspace/context/winter/cli/tracing.md`. Constraints include opt-in tracing, private attribute content,
parallel/nested context propagation, detached managed services, isolated export failure, and short linked dashboard
traces with background export. Tests include `winter-workspace/tools/winter-cli/tests/test_cli_otel_process.py`,
`winter-workspace/tools/winter-cli/tests/core/internal/test_otel_session_tracing.py`, and
`winter-workspace/tools/winter-cli/tests/modules/tui/test_dashboard_tracing.py`. The nominal legacy request timeout does
not bound encoding or slow DNS, and default diagnostic silence is not proved merely by omitting a handler; neither is
canonized here. Project constraints own global logging/tracing, avoiding duplicate policy in this initiative.
