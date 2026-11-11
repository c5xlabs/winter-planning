# Workspace rewrite baseline

## What this inventory establishes

The existing Python Workspace implementation, its tests, framework documentation, engineering conventions, and
first-party provider repositories are the evidence for this plan. They establish capabilities and regression scenarios,
not a requirement to reproduce Python internals or known defects. The replacement is a Rust Workspace component behind
the platform's executable boundary, not an in-process platform module or a Python compatibility wrapper.

All new intent remains `outline` and all epics remain `planned`. Requirements are written for review; no implementation
specs, public payload schemas, command grammar, crate decomposition, or build slices are introduced here. The platform
foundation's written specs are consumed as design input, not treated as delivered production dispatch.

The accepted interpretation of rough parity is in `decision:rough-parity`; the no-forwarding boundary is in
`decision:rust-replacement`. Historical migration proposals that forwarded unported commands into Python are not the
rewrite plan.

## Source snapshots

These are the clean source revisions examined in beta. Repository paths in the domain findings are relative to each
named code repository, not paths inside this planning repository. The planning baseline was `90dd2d7` before this work.

| Repository                       | Revision                                   | Evidence owned                                                                  |
| -------------------------------- | ------------------------------------------ | ------------------------------------------------------------------------------- |
| `winter-workspace`               | `b57b020e99e10a04d8bb621d0b03ddb2c983a090` | Python CLI, source/test behavior, launcher, framework context, lint helper      |
| `winter-context`                 | `446b54f54bd9fc8ae21efa2cbeff6d1e67d67ecd` | Existing architecture, standards, agent conventions, and verification scenarios |
| `winter-service-tmux`            | `30b8ce16227040c57c5a90034ca2566b16788113` | tmux orchestration, readiness, logging, lifecycle and recovery                  |
| `winter-service-docker`          | `3d3f692cad1155ccf64d71e4cbea10cf58c1477f` | Docker Compose isolation, status, lifecycle and logs                            |
| `winter-service-process-compose` | `e1bdf5172b59f4124f05935ac5c9cf4c4b52402b` | Per-scope instances, reconciliation, readiness and recovery                     |
| `winter-plugin-api`              | `e98191cdbb2e496c62d8e4cd38d6b961b8dd258d` | Published Python UI contribution seam and its limits                            |
| `winter-context-planning`        | `497e9e14bfd5933f7aae7a2ca3983502e0118f5d` | Planning layout, ids, requirements ownership and decision rules                 |

## Where to read the evidence

- [Core capabilities](../core-capabilities/findings.md): discovery, configuration, repository topology, worktrees,
  multi-repository Git, destructive guards, refs/locks/fingerprints, nested workspaces, source/test anchors.
- [Runtime capabilities](../runtime-capabilities/findings.md): variables, ports, provisioning, declared cleanup,
  provider semantics, all three provider implementations, failure and recovery gaps.
- [Agent capabilities](../agent-capabilities/findings.md): extension eligibility and execution, lifecycle hooks,
  projection ownership, skills, agent/model/effort resolution, external-author tooling.
- [Operator capabilities](../operator-capabilities/findings.md): doctor, lint, dependency graph, dashboard, artifact
  paths, reporting, logging and UI compatibility questions.
- [Strategy alignment](../strategy-alignment/findings.md): inherited engineering intent, current platform boundaries,
  historical proposals superseded by this request, and requirements that remain owned elsewhere.

## Capability coverage

The old commands below identify evidence, not the final replacement command grammar. An owning epic includes its
success, refusal, partial-failure, and repeat-run requirements; the project-wide result and preservation requirements
apply across the map.

| Existing surface or capability                                                                                     | Requirements owner                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------ |
| Source-local launcher, help/version, command registration, human/JSON reporting, automation boundary               | `epic:parallel-workspaces/component-runtime`; production component discovery and handoff stay in `epic:platform/launcher-shim` |
| Root lookup, shared/local overlay, raw declarations, schema/deprecation errors                                     | `epic:parallel-workspaces/workspace-configuration`                                                                             |
| `repo list/add/remove`, project/standalone roles, aliases, main branches, pin meanings, configured paths           | `epic:parallel-workspaces/repository-catalog`                                                                                  |
| Standalone ref classification, lock, `ws update`, `ws update --freeze`, `ws fingerprint`                           | `epic:parallel-workspaces/reproducible-inputs`                                                                                 |
| Workspace creation, bare `ws init`, source/standalone reconciliation, excludes, safe pruning                       | `epic:parallel-workspaces/workspace-bootstrap`                                                                                 |
| Copied framework docs and extension/project path routing; new progressive CLI context                              | `epic:agent-ready-operation/context-navigation`, with harness delivery in `epic:agent-ready-operation/harness-projection`      |
| `ws init ENV`, `--all`, list/worktrees/status/index, registry, worktree tracking initialization, destroy           | `epic:parallel-workspaces/environment-lifecycle`                                                                               |
| `ws fetch/pull/push/merge/connect/disconnect`, pattern selection and remote resilience                             | `epic:parallel-workspaces/coordinated-git`                                                                                     |
| `ws checkout/reset/clean`, preflight refusal and partial execution, untracked-file protection                      | `epic:parallel-workspaces/safe-history`                                                                                        |
| `ws restack`, frozen boundaries, ordered replay, conflict recovery                                                 | `epic:parallel-workspaces/stacked-history`                                                                                     |
| Nested root verification, recursive status, port delegation, local inheritance, loss checks and teardown           | `epic:parallel-workspaces/nested-workspaces`                                                                                   |
| `env`, workspace/feature/named bands, interpolation, command-valued entries, secret execution gates                | `epic:running-environments/scope-environment`                                                                                  |
| `provision`, declared top-level `clean`, stages/scopes/actions, required services and destroy integration          | `epic:running-environments/resource-provisioning`                                                                              |
| `service up/down/status/restart/logs`, targeting, fan-out, readiness and stream cleanup                            | `epic:running-environments/service-orchestration`                                                                              |
| tmux service maps/layouts, dependencies/probes, pane lifecycle, capture, namespace-safe cleanup                    | `epic:running-environments/provider-tmux`                                                                                      |
| Scope-pure Compose declarations, containers/ports/health, persistent data, logs and teardown                       | `epic:running-environments/provider-docker`                                                                                    |
| Per-scope process-compose instances, generated definitions, stable processes, sockets/locks and recovery           | `epic:running-environments/provider-process-compose`                                                                           |
| Hook execution, capabilities introspection/binding/version checks, `ext scaffold/verify`, executable contributions | `epic:agent-ready-operation/executable-extensions`                                                                             |
| Extension discovery, manifests, enable/disable/adoption, identity, source selection and activation                 | `epic:agent-ready-operation/extension-management`                                                                              |
| Skills and agent copies, vendor transforms, context hook, exclusions, managed ownership and retraction             | `epic:agent-ready-operation/harness-projection`                                                                                |
| `agents`, canonical definitions, tiers, overrides, effort, vendor permissions, effective provenance/drift          | `epic:agent-ready-operation/agent-model-resolution`                                                                            |
| `doctor`, health probes, external diagnostics, remediation and warnings/failures                                   | `epic:agent-ready-operation/workspace-diagnostics`                                                                             |
| `lint`, changed-set selection, extractability/size/service checks, contributions, audited ignores                  | `epic:agent-ready-operation/context-quality`                                                                                   |
| `graph`, sorted declared module dependencies and incomplete-manifest behavior                                      | `epic:agent-ready-operation/dependency-graph`                                                                                  |
| `dashboard`, layouts/drilldown/commit graph/model matrix, actions/keybindings, errors and UI contributions         | `epic:agent-ready-operation/workspace-dashboard`                                                                               |
| `space KIND`, pure configured artifact-directory resolution                                                        | `epic:agent-ready-operation/artifact-space`                                                                                    |
| Opt-in tracing, diagnostic privacy, output separation and external-process policy                                  | Project `engineering.md`, with capability-specific bounds in the owning epics                                                  |
| Existing-workspace conversion, PATH conflicts, compatibility notes, rollout proof and artifacts                    | `initiative:rust-adoption`                                                                                                     |

## Work that can proceed independently

These are capability dependencies, not implementation slices. Epic requirements own the actual prerequisites; this table
explains why the whole rewrite need not be a single serial port.

| Track                      | Starts from                                                                           | Integration boundary                                                                                                     |
| -------------------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Product foundation         | Platform foundation design and direct local Workspace invocation                      | Installed handoff waits for Platform's production contract; local development does not wait for releases                 |
| Declaration and read paths | Workspace configuration and repository inventory                                      | Artifact paths, dependency queries, reproducibility and context can be developed without starting services               |
| Feature and Git safety     | Repository catalog and authoritative environment identity                             | Ordinary Git coordination, destructive-history guards, and stacked-history replay have separate acceptance               |
| External contributions     | Component process boundary and configuration                                          | Executable contract work can proceed beside Git; activation, models, and projection integrate on that boundary           |
| Runtime isolation          | Environment identity and variable resolution                                          | Provision stages and service control can be developed separately; required-service and destroy flows join them           |
| First-party providers      | Shared service semantics and conformance fixtures                                     | tmux, Docker Compose, and process-compose are independently deliverable and testable tracks                              |
| Operator tools             | Shared local queries and installed-contribution models                                | Diagnostics/lint/graph can grow without the dashboard; dashboard is a consumer, not the only interface                   |
| Nested composition         | Root verification, environment identity, runtime isolation and teardown results       | Child state must be complete enough for destructive preflight, not merely suitable for a display                         |
| Adoption and delivery      | Preview planning can start early; acceptance accumulates across all capability tracks | Migration/cutover and full-parity release wait for reviewed differences, provider coverage and shared release enablement |

## Differences that must not disappear into the port

The evidence includes implementation gaps. Refusal and recovery behavior are requirements to settle, not accidental
compatibility obligations: corrupt registry state becoming empty, conflicting environment-discovery models, implicit
provider checks weaker than explicit checks, missing service status counting as ready, shared cleanup repeated per
environment, prefix-only artifact ownership, and UI callbacks tied to Python objects.

The domain findings retain source/test anchors and the owning decisions retain unresolved choices. A future spec must
resolve the decisions that bind its epic; it cannot treat this inventory as approval for every existing inconsistency.
No runtime verification was performed against beta or another live environment during this investigation.
