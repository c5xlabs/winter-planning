# Agent integration capability findings

Read-only source, test, and context inspection of the existing Python Workspace CLI; no legacy CLI commands, hooks,
contributed checks, or providers were executed. This is an evidence map, not a protocol or implementation specification.

Evidence paths are repository-relative: `winter-workspace/` names the existing code repository and `winter-context/`
names its existing conventions repository. `CLI` below abbreviates `winter-workspace/tools/winter-cli/`.

## Capability map

| Existing surface                                                                                       | Planning owner                                                                                                                                                                                         | Source and test evidence                                                                                                                                                                                                                                                        |
| ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Executable hooks; cwd, managed environment, streamed output, strict teardown                           | `epic:agent-ready-operation/executable-extensions`                                                                                                                                                     | `CLI/src/winter_cli/modules/workspace/extension_hook_service.py`; `init_service.py`; `destroy_service.py`; `CLI/tests/modules/workspace/test_extension_hook_service.py`; `test_init_service.py`                                                                                 |
| Provider candidates, explicit/implicit lists, declared implementation versions                         | `epic:agent-ready-operation/executable-extensions`                                                                                                                                                     | `CLI/src/winter_cli/modules/capability/capability_registry_service.py`; `version_compat.py`; `CLI/tests/modules/capability/test_capability_registry_service.py`                                                                                                                 |
| Capability scaffold and golden invocation checks                                                       | `epic:agent-ready-operation/executable-extensions`                                                                                                                                                     | `CLI/src/winter_cli/modules/ext/{scaffold_service,verify_service}.py`; `CLI/src/winter_cli/modules/capability/specs/service-v1.toml`; `CLI/tests/modules/ext/{test_scaffold_service,test_verify_service}.py`                                                                    |
| Python plugin discovery, registration, badges, actions, panels, arbitrary screens                      | Execution boundary: `epic:agent-ready-operation/executable-extensions`; contribution scope: `epic:agent-ready-operation/workspace-dashboard`                                                           | `CLI/src/winter_cli/plugins/{loader,types}.py`; `plugins/internal/importlib_plugin_loader.py`; `CLI/tests/plugins/test_loader.py`; `winter-context/architecture/plugin-author.md`                                                                                               |
| Manifest fields, standalone/project eligibility, duplicate declarations, opt-out/context-only adoption | `epic:agent-ready-operation/extension-management`                                                                                                                                                      | `CLI/src/winter_cli/modules/workspace/{extension_manifest,repository_factory}.py`; `CLI/tests/modules/workspace/{test_extension_manifest,test_repository_factory}.py`; `test_init_service.py`                                                                                   |
| Shared/local configuration, refs, locks, comment-preserving edits                                      | `epic:parallel-workspaces/workspace-configuration`, `epic:parallel-workspaces/repository-catalog`, `epic:parallel-workspaces/reproducible-inputs`; activation consumes them                            | `CLI/src/winter_cli/config/{workspace,overlay}.py`; `util.py`; `config/internal/write_winter_configuration_repository.py`; `modules/workspace/internal/config_lock_repository.py`; `CLI/tests/config/test_overlay.py`                                                           |
| Extension config/assets and managed excludes                                                           | `epic:agent-ready-operation/extension-management` with `epic:agent-ready-operation/artifact-space`                                                                                                     | `CLI/src/winter_cli/core/extension_invocation.py`; `modules/workspace/extension_exclude_service.py`; `internal/managed_block.py`; `CLI/tests/core/test_extension_invocation.py`; `CLI/tests/modules/workspace/test_extension_exclude_service.py`                                |
| Eager/lazy context, candidate entrypoints, feature-worktree routing                                    | Source/address ownership: `epic:agent-ready-operation/context-navigation`; eligibility: management; delivery: projection                                                                               | `CLI/src/winter_cli/modules/workspace/extension_agentsmd_service.py`; `CLI/tests/modules/workspace/test_extension_agentsmd_service.py`; `winter-context/agent-context/references.md`                                                                                            |
| Skill links/copies, workspace skills, OpenCode name transform, pruning                                 | `epic:agent-ready-operation/harness-projection`                                                                                                                                                        | `CLI/src/winter_cli/modules/workspace/{extension_skill_install,extension_symlink_service,workspace_skill_service}.py`; `CLI/tests/modules/workspace/{test_extension_symlink_service,test_workspace_skill_service}.py`                                                           |
| Agent rendering, source-link migration, byte comparison, copy drift                                    | `epic:agent-ready-operation/harness-projection`                                                                                                                                                        | `CLI/src/winter_cli/modules/workspace/agent_install.py`; `agent_transform/{renderers,agent_copy_inspector}.py`; `CLI/tests/modules/workspace/{test_extension_agent_service,test_agent_copy_inspector}.py`                                                                       |
| Canonical agents; model tiers, concrete overrides, effort and provenance                               | `epic:agent-ready-operation/agent-model-resolution`                                                                                                                                                    | `CLI/src/winter_cli/modules/workspace/agent_transform/{canonical_parser,model_tiers,renderers}.py`; `modules/agents/agent_matrix_service.py`; `CLI/tests/modules/workspace/{test_agent_model_overrides,test_agent_render_bytes}.py`                                             |
| Diagnostic/lint executables, NDJSON, scope, suppression and dependency rules                           | Contributions: management/execution; policy/reporting: `epic:agent-ready-operation/workspace-diagnostics`, `epic:agent-ready-operation/context-quality`, `epic:agent-ready-operation/dependency-graph` | `CLI/src/winter_cli/modules/{doctor/extension_probe_service,doctor/probe_parser,lint/extension_lint_service,lint/finding_parser,lint/ignore_service,graph/graph_service}.py`; `winter-workspace/tools/winter-lint/extractability.py`; `winter-context/agent-context/linting.md` |

## Observed contracts worth preserving

- Standalone wins a same-name project/standalone declaration; either declaration's extension opt-out excludes
  activation. Project contributions use the source checkout; explicit manifest-less project context is context-only.
- Prefix precedence is workspace override, manifest prefix, manifest name, repository name. Skills use a bare prefix
  when their directory equals it; other skills and all agent files gain a prefix. Workspace skills remain independent of
  extension adoption.
- Claude/Codex skill directories are relative symlinks; OpenCode `.opencode/skill` holds real copies with installed-name
  frontmatter. Agent targets are `.claude/agents/*.md`, `.codex/agents/*.toml`, `.opencode/agent/*.md`.
- Agent identity is canonical frontmatter name for Claude/Codex but prefixed filename for OpenCode. Codex body becomes
  `developer_instructions`; OpenCode defaults to subagent mode. Other vendors' override blocks do not project.
- Model precedence is workspace agent override, canonical harness override, configured tier, built-in default. Bare
  override strings are tiers; per-vendor strings/profile models are always concrete, even `haiku`. Effort resolves
  separately; effort-only profiles preserve the model and absent effort inherits.
- Shared/local merging is one-level per selected table; local agent/tier entries replace whole entries, not vendor
  cells. Built-in tier overlay then merges vendor cells. Resolution records shared/local provenance, not just final
  strings.
- All vendor agent renders finish before the first write. A real-filesystem regression protects canonical source when a
  legacy agent symlink is replaced: `CLI/tests/modules/workspace/test_extension_agent_service.py:497-531`.
- Workspace hooks run once after workspace reconciliation and before the all-environment loop. Environment hooks use
  environment cwd and persisted identity; destroy hooks precede outer removal. Strict destroy stops removal after hook
  failure; non-strict currently continues and deliberately excludes hook failure from its final exit status.
- Doctor/lint parse each NDJSON line; malformed lines warn, nonzero exit appends failure without discarding findings.
  Lint scripts receive selected paths and the launching CLI path. Suppression is post-hoc, unions repo/workspace rules,
  retains rule provenance, and cannot conceal a check crash. Scoped worktree ignores come from that worktree, while lint
  executables come from the source/installed extension.
- `requires` is a content dependency declaration and graph input, not transitive installation or hook ordering. The
  extractability linter uses selected-checkout declarations plus the installed global graph and skips marked examples.

## Current inconsistencies and redesign boundaries

- `adopt_extensions = none` gates projection, hooks, context, and extension checks, but not graph/capability consumers
  or standalone Python plugin discovery. Eligibility is not currently one authoritative activation policy.
- Ownership is inferred from prefixes. Agent files and copied skill directories can overwrite/prune user content inside
  those prefixes; removed extensions or changed prefixes lack reliable old ownership. Retraction avoids overlapping live
  prefixes but cannot reconstruct missing or invalid declarations. Config collision validation is not a complete
  resolved namespace preflight.
- Invalid agent renders are skipped but omitted from the live-name set, so prior good copies can then be pruned. Render
  preflight prevents render-time half-installation, not I/O partial writes; copied skills use delete-then-copy. Existing
  lock upserts protect threads in one process, not competing CLI processes.
- `extension_agentsmd_service.py` falls back to repo-kind context defaults after malformed manifests; a standalone can
  become eager. `init_service.py` returns early with no repositories and reconciles excludes only, bypassing context
  cleanup. Project-kind inference uses a `projects/` path shape rather than explicit provenance.
- `internal/managed_block.py` treats an unterminated block as extending to EOF. Exclude writes normally preserve outside
  text and use Git exclude files, despite an older documentation example claiming tracked `.gitignore` edits.
- `doctor/skill_probe_service.py` computes extension names without the installer's bare-prefix exception and does not
  compare actual link targets to expected source targets. Installer and drift semantics must agree, not inherit this
  gap.
- Explicit capability bindings check implementation versions; implicit paths do not. Implicit multi-provider resolution
  can also bypass entrypoint validation. Current registry tests cover explicit incompatible versions, not symmetric
  enforcement of every selection path.
- `ext/verify_service.py` invokes real action words and treats exit other than 2 as action recognition; it is not
  isolated behavioral proof. It builds four environment variables independently, misses the shared service-prefix
  builder and configured asset-directory override, selects highest versions lexically, and can accept parseable describe
  output without a successful exit. The bundled spec has describe/catalog beyond the older usage prose.
- The scaffold is currently Python; this is an author example, not a Rust support obligation. First-party behavior
  cannot retain Python through scaffold validation or bundled lint delegation under the replacement decision.
- `plugins/loader.py` imports arbitrary Python from workspace, user-global, then standalone paths; failed
  higher-priority loading still shadows fallback. Its contribution objects depend on concrete runtime models and
  arbitrary UI objects. Collected Click commands are not attached to dispatch. The documented narrow external plugin
  package is not proof the runtime seam implements those views or a language-neutral API.
- Top-level canonical unknown fields are not uniformly rejected; runtime defaults are looser than authoring guidance.
  Native permission-key presence suppresses tools-loss warnings without proving equivalent restrictions. Per-vendor
  model strings are concrete in source and a dedicated test despite contradictory configuration prose.

## Trust and execution limits

Resolved containment and executable checks exist for hooks and diagnostic scripts, but not uniformly for prefixes,
canonical names, source roots, and capability entrypoints. Hook invocation inherits the parent environment; workspace
hooks do not scrub inherited feature-scope variables. Init hooks run after attempted repository work even when it
failed, and dependency declarations do not order hooks or guarantee installed prerequisites.

The subprocess runner has tracing, non-interactive modes, and optional process-group cleanup, but ordinary hooks do not
request managed groups or finite deadlines. Declarative inspection is distinguishable from doctor/lint/conformance,
which can execute arbitrary external programs. No current behavior proves sandboxing or reversibility of their effects.

The rewrite therefore needs a versioned external execution boundary with bounded policies and explicit authorization,
not in-process import compatibility. Shared Platform protocol ownership remains unresolved; provider action semantics
stay with service orchestration, dashboard contribution scope stays with the dashboard, and last-good retention,
ownership recovery, cross-process exclusion, and invalid-manifest behavior remain owned open decisions in the four
epics.
