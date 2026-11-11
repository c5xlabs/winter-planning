# Workspace configuration decisions

## Root selection (decision:parallel-workspaces/workspace-configuration/root-selection)

**Decider:** product owner and engineering

**Open.** Choose nearest-root discovery, explicit root selection, and ambiguity handling for nested roots, symlinks, and
outside-workspace invocations; this blocks finalizing `product:identify-workspace`, not its no-source-checkout promise.

## Layering policy (decision:parallel-workspaces/workspace-configuration/layering-policy)

**Decider:** product owner and engineering

**Open.** Choose which local lists append, replace, or override by identity and how nested tables merge, rather than
silently inheriting the old one-level merge; this blocks finalizing `product:effective-configuration` and the relevant
conversion choices under `decision:configuration-transition`.

## Validation boundary (decision:parallel-workspaces/workspace-configuration/validation-boundary)

**Decider:** product owner and engineering

**Open.** Choose strict versus extensible configuration areas and which invalid optional definitions block which
commands; this blocks finalizing `product:configuration-validation` and diagnostic compatibility, not refusal before
unsafe mutation.

## Concurrent edits and recovery (decision:parallel-workspaces/workspace-configuration/write-conflicts)

**Decider:** engineering and product owner

**Open.** Choose conflict reporting, retry, and operator recovery for concurrent or interrupted configuration writes,
including edits by external tools; this blocks finalizing `product:configuration-state-ownership` and
`engineering:configuration-write-integrity` without selecting a file format or storage library.
