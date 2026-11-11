# Core Workspace capability provenance

Read-only investigation of the current Python Workspace source, tests, and guides. Tests were inspected, not run; this
is evidence for capability coverage, not a declaration that existing quirks are the Rust requirements. Capability
preservation follows `decision:rough-parity`; native replacement follows `decision:rust-replacement`. Command names
below are legacy names, not the vocabulary selected by `decision:command-vocabulary`.

Paths are relative to the `winter-workspace` code repository. `S/` abbreviates `tools/winter-cli/src/winter_cli/`, `T/`
abbreviates `tools/winter-cli/tests/`, and `D/` abbreviates `context/winter/`.

## Legacy command to capability owner

| Old surface                                                       | Requirements owner                                             |
| ----------------------------------------------------------------- | -------------------------------------------------------------- |
| Root discovery, shared/local config loading                       | `epic:parallel-workspaces/workspace-configuration`             |
| `repo list`, `repo add`, `repo remove`                            | `epic:parallel-workspaces/repository-catalog`                  |
| `winter init`, bare `ws init`, `ws prune`                         | `epic:parallel-workspaces/workspace-bootstrap`                 |
| `ws update`, `--freeze`, ref/lock, `ws fingerprint`               | `epic:parallel-workspaces/reproducible-inputs`                 |
| `ws init ENV`, `ws destroy`, `ws list`, `ws index`                | `epic:parallel-workspaces/environment-lifecycle`               |
| `ws init --all`                                                   | Bootstrap scope; environment-lifecycle mechanics               |
| `ws worktrees`                                                    | Catalog paths; lifecycle env identity; coordinated-git status  |
| `ws connect`, `ws disconnect`, `ws diff`                          | `epic:parallel-workspaces/coordinated-git`                     |
| `ws fetch`, `ws pull`, `ws merge`, `ws push`                      | `epic:parallel-workspaces/coordinated-git`                     |
| `ws status`, `ws status --fetch`                                  | Coordinated-git snapshot/freshness; diagnostics broader health |
| `ws checkout`, `ws reset`, `ws clean`                             | `epic:parallel-workspaces/safe-history`                        |
| `ws restack`                                                      | `epic:parallel-workspaces/stacked-history`                     |
| Nested init/status/destroy and child workspace-service stop       | `epic:parallel-workspaces/nested-workspaces`                   |
| `env SCOPE`, env bands and command-valued entries                 | `epic:running-environments/scope-environment`                  |
| `provision`, top-level `clean`, provision teardown during destroy | `epic:running-environments/resource-provisioning`              |
| Required-services checks and runtime service control              | `epic:running-environments/service-orchestration`              |
| `doctor` registry/port/config checks                              | `epic:agent-ready-operation/workspace-diagnostics`             |
| `space KIND`                                                      | `epic:agent-ready-operation/artifact-space`                    |

## Source and test anchors

- Configuration/discovery: `S/config/workspace.py`, `S/config/overlay.py`, `S/util.py`,
  `S/config/internal/cwd_workspace_locator.py`, and `tools/winter-cli/bin/winter`.
  `T/config/internal/test_cwd_workspace_locator.py::test_find_workspace_root_ignores_env_local_winter_logs_dir` proves a
  log-only marker is not a root. `T/config/test_workspace_config_service.py` proves array append and whole-band local
  replacement, duplicate standalone-name refusal, nested-key validation, and deferred optional parsing.
- Inventory/bootstrap: `S/modules/workspace/repository_factory.py`, `S/modules/workspace/handlers/repo_handler.py`,
  `S/modules/workspace/init_service.py`, and `S/modules/workspace/prune_service.py`.
  `T/modules/workspace/test_init_service.py` covers incidental Git autotracking, unpushed upstream inference, bootstrap
  despite tracking failure, and continuing after an earlier env failure.
- Workspace-linked-worktree excludes: `S/modules/workspace/internal/gitpython_workspace_exclude_locator.py` and
  `T/modules/workspace/internal/test_gitpython_workspace_exclude_locator.py`; real Git cases cover sibling isolation, a
  bare common directory, idempotent setup, and one sibling's destroy leaving another's excludes intact.
- Reproducibility: `S/modules/workspace/workspace_sync_service.py`,
  `S/modules/workspace/workspace_fingerprint_service.py`, `S/modules/workspace/internal/config_lock_repository.py`, and
  `S/modules/workspace/internal/gitpython_repository.py`. Tests in `T/modules/workspace/test_ws_update.py`,
  `T/modules/workspace/test_ws_update_freeze.py`, and `T/modules/workspace/test_workspace_fingerprint_service.py` cover
  held pins, local-layer freezing, locally-only commits, index-byte preservation, staged/working differences, and
  same-size edits within one timestamp tick. `T/config/internal/test_write_config_lock_repository.py` covers concurrent
  in-process upsert and malformed/versioned locks.
- Environment identity: `S/modules/workspace/env_index.py`, `S/modules/workspace/internal/toml_env_index_registry.py`,
  `S/modules/workspace/internal/read_workspace_repository.py`, `S/modules/workspace/init_service.py`, and
  `S/modules/workspace/destroy_service.py`. `T/modules/workspace/test_env_index_allocator.py` covers collisions,
  wraparound, reserved slots, and exhaustion; `T/modules/workspace/test_phase3_registry_lifecycle.py` covers durable
  assignment and dry-run release avoidance.
- Bulk Git: `S/modules/workspace/workspace_sync_service.py`, `S/modules/workspace/workspace_push_service.py`,
  `S/modules/workspace/workspace_merge_service.py`, `S/modules/workspace/pattern_match.py`,
  `S/modules/workspace/ref_resolver.py`, `S/modules/workspace/internal/read_repo_repository.py`,
  `S/modules/workspace/internal/write_repo_repository.py`, and `S/modules/workspace/internal/git_ops_service.py`.
  Service tests prove per-worktree upstreams independent of repo order, best-effort source sync, held pins, and
  successful remote push despite failed local follow-up.
  `T/modules/workspace/internal/test_write_repo_repository_local_ff.py` covers checked-out, unchecked-out, dirty, and
  out-of-sync landing branches.
- Destructive Git: `S/modules/workspace/worktree_safety.py`, `S/modules/workspace/env_checkout_service.py`,
  `S/modules/workspace/env_reset_service.py`, and `S/modules/workspace/env_clean_service.py`.
  `T/modules/workspace/test_env_checkout_service_detached_head.py` proves protection of the env branch behind detached
  HEAD. `T/modules/workspace/internal/test_write_repo_repository.py` proves empty-directory preview/removal parity,
  ignored-file preservation, and actual removed-path reporting after a partial clean failure.
- Stacks: `S/modules/workspace/env_restack_plan_service.py` and `S/modules/workspace/env_restack_service.py`;
  `T/modules/workspace/test_env_restack_plan_service_real_git.py` and
  `T/modules/workspace/test_env_restack_service_real_git.py` prove reflog boundaries after amend, same-named-tag
  isolation, conflict/continue/rerun, cut continuation, and preservation of exactly the dependant's commit subjects
  rather than merely successful outcome enums.
- Nesting: `S/modules/workspace/nested_workspace_service.py`, `S/modules/workspace/nested_overlay.py`,
  `S/modules/workspace/nested_env.py`, `S/modules/workspace/nested_status.py`, and
  `S/modules/workspace/internal/subprocess_nested_workspace_runner.py`.
  `T/modules/workspace/test_nested_workspace_service.py` proves wrong-root refusal and recursion-fanout prevention;
  `T/modules/workspace/test_nested_workspace_e2e.py` covers real cloning, inherited identity, delegated ports, retained
  branch/stash refusal, and acceptance of pushed-but-unmerged work. `T/modules/workspace/test_destroy_service.py` covers
  child shutdown failure preserving the outer worktree.
- Runtime handoffs: `S/modules/workspace/env_provisioner.py`, `S/modules/workspace/env_band_resolver_service.py`,
  `S/modules/provision/manifest.py`, `S/modules/provision/provision_service.py`,
  `S/modules/provision/execution_service.py`, and `S/modules/provision/service_check_service.py`.
  `T/modules/workspace/test_env_band_resolver_service.py` covers command masking/import limits and output provenance;
  `T/modules/provision/test_provision_service.py` covers ordering, clean best-effort, selectors, reset fallback, and
  required-service checks. These belong to runtime requirements, not duplicated lifecycle implementations.
- Operational intent: `D/environment-lifecycle.md`, `D/worktree-ops.md`, `D/workspace-layout.md`,
  `D/cli/configuration/config-files.md`, `D/cli/configuration/repositories.md`,
  `D/cli/configuration/ports-and-environments.md`, and commands under `D/cli/usage/ws/`.

## Observed discrepancies requiring deliberate choices

- Launcher discovery needs config plus CLI source, whereas Python discovery needs only config. Creation must no longer
  depend on the source-local launcher's empty-config bootstrap.
- Ordinary env enumeration uses aliases plus registry; reconcile-all uses the first project's worktree list; doctor
  scans worktree-pointer children. Manual, partial, or missing-first-project envs can disagree across these views.
- Config merge appends repo arrays but only shallow-merges selected tables; malformed settings mix strict refusal,
  fallback, and skipping. The requirements retain explainability, not those accidental inconsistencies.
- State-registry corruption is treated as an empty document; writes assume one allocator. Lock upserts coordinate only
  one process's threads. Neither is sufficient evidence of cross-process safety.
- Alias allocation precedes existing-assignment lookup; changing aliases can move a supposedly stable index. Nested
  slices depend on configuration order, and neither parent service offsets nor host port use are fully confined.
- A fresh branch lock invokes branch checkout against current origin state, not explicitly the locked commit. Branch
  reproducibility and moving-pin semantics therefore need a decision rather than a claim of exact legacy parity.
- Fetch advances source checkout state. Pull warns and can integrate stale refs after failed source refresh. Existing
  source checkout branch position is assumed, not comprehensively established before that advancement.
- Checkout/reset are all-or-nothing at preflight, not at execution. Checkout has no preview flag. Git clean preserves
  partial deletion evidence, but its preview does not freeze the later removal set.
- Destroy can continue whole-directory removal and index release after worktree removal failure. Prune's clean/worktree
  checks do not audit every local-only commit or stash. These gaps are not preservation guarantees to reproduce.
- Literal/zero-match behavior and prompting differ across commands; structured mode sometimes emits human no-match text
  or requires prompt handling. Output compatibility is left to `decision:command-vocabulary`.
- Normal status tolerates and omits some standalone probe failures; nested deletion consumes status as safety evidence.
  Destructive completeness must not equate omitted or failed probes with no work.
- Provision-before-run is documented but not a demonstrated persisted readiness gate. Top-level clean runs arbitrary
  shell commands without confirmation, and workspace-scoped handlers run at the shared root once per matched env.
- Command-valued env output has known masked-import and delayed-output collision limitations; shell substitution safety
  does not remove ordinary environment-variable channels or argv splitting after unquoted substitution.

## Sequencing and limits

Configuration follows component runtime; catalog follows configuration. Reproducible inputs follow catalog. Bootstrap
follows catalog and context navigation, not environment lifecycle or harness projection; it places the minimum common
eager safety hook in `AGENTS.md` and the `CLAUDE.md` shim while preserving user content. Environment lifecycle follows
catalog, coordinated Git follows lifecycle, safe history follows coordinated Git, and stacked history follows safe
history. Nested workspaces additionally need scope environment, provisioning, and service orchestration; wire those
later without making core lifecycle or runtime depend back on nesting.

This investigation did not choose Rust libraries, storage formats, result schemas, a child protocol, or final command
names. It did not exercise live environments, run tests, inspect provider implementations exhaustively, or establish
cross-platform Git behavior. Those limits are not permission to omit the capabilities; unresolved choices live with the
owning epics, and existing-workspace conversion/parity/release remain in `initiative:rust-adoption`.
