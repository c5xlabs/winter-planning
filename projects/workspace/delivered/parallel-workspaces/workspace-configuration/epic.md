---
epic: parallel-workspaces/workspace-configuration
refinement: outline
status: delivered
---

# Know which workspace and configuration will be used

Part of `initiative:parallel-workspaces`; product requirements live in [requirements.md](./requirements.md). Supports
the other parity epics through one identifiable workspace and one effective configuration that every command reads the
same way.

## The problem

Engineers and agents can invoke a tool from a worktree, nested workspace, or unrelated directory and mistake which
configuration it will use. Local overrides and invalid declarations can silently change the operation they intended.

## The outcome

Every command addresses the workspace the reference tool would address from the same directory, reads the same shared
and local settings with the same precedence, and refuses the same invalid declarations before it has any effect. A
declaration on which the reference tool fails without explanation is refused with a message that names the file or the
key. Workspace discovery does not depend on carrying tool source.

At the end of this epic the two commands the binary has, `ws index` and `repo remove`, run against any workspace
configuration: each loads and validates the whole configuration as the reference tool does, so every refusal of the
configuration model can be seen by running one of them in a scratch workspace.

## What this epic is not

- A command that shows, explains, validates, or edits configuration as such. The reference tool has none, and none is
  added; `decision:parallel-workspaces/workspace-configuration/configuration-inspection` holds the question.
- The launcher's own rules: its second marker for a workspace root, `init`, and `--winter=PATH` are absent under
  `decision:parallel-workspaces/component-runtime/launcher-only-surface`.

## Depends on

- `epic:parallel-workspaces/component-runtime`

## Left for later

- What the repository entries mean beyond their load-time rules, and the commands that add, remove, list, and pin
  repositories, belong to `epic:parallel-workspaces/repository-catalog` and
  `epic:parallel-workspaces/reproducible-inputs`; the lock file is specified here and first read and written there.
- Env index allocation and release, and the writes of the machine-local state file, belong to
  `epic:parallel-workspaces/environment-lifecycle`; the file is specified here and `ws index` reads it.
- The keys a nested workspace is handed, and the writer that hands them over, belong to
  `epic:parallel-workspaces/nested-workspaces`.
- What each optional table means once loaded belongs to the epic that owns the feature: `[env]` to
  `epic:running-environments/scope-environment`, `[capabilities]` and `[[service]]` to
  `epic:running-environments/service-orchestration`, `[provision]` to `epic:running-environments/resource-provisioning`,
  `[model_tiers]` and `[agent_model_overrides]` to `epic:agent-ready-operation/agent-model-resolution`, `[space]` to
  `epic:agent-ready-operation/artifact-space`, `lint` to `epic:agent-ready-operation/context-quality`, `doctor` and the
  configuration probes to `epic:agent-ready-operation/workspace-diagnostics`, `[tui]` and `[keybindings]` and the reload
  on every refresh to `epic:agent-ready-operation/workspace-dashboard`.
- Existing-workspace conversion is not planned: `decision:configuration-transition` keeps the reference tool's files.
