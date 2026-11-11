# Workspace configuration requirements

For `epic:parallel-workspaces/workspace-configuration`.

## Identify the workspace before acting (product:identify-workspace)

A person or agent can tell which workspace an invocation addresses, independently of the component executable selected
to run it. Selection policy is settled by `decision:parallel-workspaces/workspace-configuration/root-selection`.

Acceptance:

- Invocation from supported workspace subdirectories identifies the workspace root used by the operation.
- Per-environment logs and state do not masquerade as workspace roots; a genuine nested workspace is distinguishable.
- Missing or ambiguous selection reports the location considered and how to select the intended workspace.
- Selecting alternate component code never implies a scratch workspace or permission to change a different target.
- An initialized workspace remains discoverable without a checkout of Winter source or copied framework documentation.

## Understand shared and local settings (product:effective-configuration)

An engineer can combine committed team settings with private local settings and predict which values apply. The merge
choices remain open under `decision:parallel-workspaces/workspace-configuration/layering-policy`.

Acceptance:

- Supported shared and local declarations have documented precedence, including lists, nested tables, and repo entries.
- The caller can distinguish an inherited value, a local replacement, a default, and machine-generated state.
- Local repository additions do not silently remove team declarations or create an unexplained duplicate identity.
- Reading effective settings does not run commands embedded in those settings or materialize missing repositories.

## Inspect settings without exposing secrets (product:configuration-explanation)

The engineer can explain an operation's effective settings and their source without printing private values by default.

Acceptance:

- Inspection identifies the workspace and contributing configuration layers, including defaults where relevant.
- Invalid or unsupported settings identify their source and subject rather than appearing as an unrelated command error.
- Secret-bearing values and command-derived values are not resolved or printed by ordinary configuration inspection.
- Inspection does not rewrite configuration, allocate an environment index, or create a local overlay.

## Correct invalid declarations before effects (product:configuration-validation)

An engineer receives actionable diagnoses for invalid declarations without an unrelated malformed optional feature
making every command unusable. Validation boundaries are settled by
`decision:parallel-workspaces/workspace-configuration/validation-boundary`.

Acceptance:

- Invalid operation-relevant configuration refuses the operation before its mutations or external commands begin.
- Unknown keys and unsupported value types are diagnosed according to an explicit policy, not accidentally ignored.
- Removed or incompatible settings name the required correction or migration instead of silently changing meaning.
- An invalid optional feature remains inspectable and does not silently substitute a default that authorizes effects.

## Keep configuration and machine state distinct (product:configuration-state-ownership)

An engineer can share a workspace definition without sharing another machine's allocations or losing private settings.

Acceptance:

- Shared definitions, local overrides, committed reproducibility records, and machine-local state have distinct roles.
- A supported version change either reads existing input with stated meaning or requests explicit conversion under
  `decision:configuration-transition`; inspection never performs conversion implicitly.
- A missing optional local layer differs from an unreadable or corrupt existing layer; corruption is not an empty file.
- Concurrent supported configuration edits preserve unrelated changes or report a conflict rather than losing one.
