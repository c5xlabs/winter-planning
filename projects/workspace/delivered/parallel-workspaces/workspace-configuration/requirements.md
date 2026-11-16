# Workspace configuration requirements

For `epic:parallel-workspaces/workspace-configuration`.

## Identify the workspace before acting (product:identify-workspace)

A person or agent can tell which workspace an invocation addresses, independently of the component executable selected
to run it. Selection policy is settled by `decision:parallel-workspaces/workspace-configuration/root-selection`.

Acceptance:

- Invocation from any directory of a workspace addresses the nearest workspace root at or above that directory.
- Per-environment logs and state do not masquerade as workspace roots; a genuine nested workspace is distinguishable.
- A missing workspace is reported with the directory the search started from and the file it looked for. The working
  directory is the only way to select a workspace.
- Selecting alternate component code never implies a scratch workspace or permission to change a different target.
- An initialized workspace remains discoverable without a checkout of Winter source or copied framework documentation.

## Understand shared and local settings (product:effective-configuration)

An engineer can combine committed team settings with private local settings and predict which values apply. The merge
rules are settled by `decision:parallel-workspaces/workspace-configuration/layering-policy`.

Acceptance:

- Supported shared and local declarations have documented precedence, including lists, nested tables, and repo entries.
- Any key may be set in the local file; a key removed from the product is refused from either file.
- Local repository entries are added after the shared ones and never replace or change one; two standalone repositories
  that resolve to one name are refused wherever they are declared. A repository key that is not a list is refused, in
  either file, under `decision:parallel-workspaces/workspace-configuration/repository-array-type`.
- Reading effective settings does not run commands embedded in those settings or materialize missing repositories.

## Inspect settings without exposing secrets (product:configuration-explanation)

**Proposed.** The engineer can explain an operation's effective settings and their source without printing private
values by default. The reference tool has no such command, and whether one is added stays open under
`decision:parallel-workspaces/workspace-configuration/configuration-inspection` until
`epic:agent-ready-operation/context-navigation` is refined.

Acceptance:

- Inspection identifies the workspace and contributing configuration layers, including defaults where relevant.
- Secret-bearing values and command-derived values are not resolved or printed by ordinary configuration inspection.
- Inspection does not rewrite configuration, allocate an environment index, or create a local overlay.

## Correct invalid declarations before effects (product:configuration-validation)

An engineer is told which declaration is invalid before a command acts on it. Validation boundaries are settled by
`decision:parallel-workspaces/workspace-configuration/validation-boundary`.

Acceptance:

- An invalid configuration refuses every command that needs the workspace, before its mutations or external commands
  begin; help, the version, and usage errors need no workspace and are not refused.
- The refusal names the key at fault, and for a repository entry the repository. A file that cannot be read or parsed is
  named by its path.
- Unknown keys are ignored without a message, except in the few tables whose keys are a closed set.
- A removed setting is refused with the name of the setting that replaces it.
- A setting documented as falling back takes its default, without a message, when its value has the wrong type.
- A text field of a repository entry whose value is not text is refused, whatever the value, under
  `decision:parallel-workspaces/workspace-configuration/element-type-validation`.
- `[provision]` and `[[service]]` are checked only by the commands that use them.

## Keep configuration and machine state distinct (product:configuration-state-ownership)

An engineer can share a workspace definition without sharing another machine's allocations or losing private settings.

Acceptance:

- Shared definitions, local overrides, committed reproducibility records, and machine-local state have distinct roles.
- The files of an existing workspace are read and written as they are, with no conversion, under
  `decision:configuration-transition`.
- A missing local file is no error; one that cannot be read or is not valid TOML is a configuration error. A path that
  is not a regular file counts as missing. The machine-local `state.toml` is a configuration error when it exists and
  cannot be read, is not UTF-8, or is not valid TOML, under
  `decision:parallel-workspaces/workspace-configuration/state-read-failures`; an absent one is an empty registry.
- An edit the tool makes keeps the comments, order, and unrelated content of the file, and its line endings and
  encoding, under `decision:parallel-workspaces/workspace-configuration/write-byte-preservation`.
- Concurrent supported configuration edits preserve unrelated changes; they are serialised, not reported as conflicts,
  under `decision:parallel-workspaces/workspace-configuration/write-conflicts`.
- A configuration file that cannot be replaced safely is refused and left unchanged, under
  `decision:parallel-workspaces/workspace-configuration/replacement-edges`.
