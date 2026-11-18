# Workspace configuration engineering requirements

For `epic:parallel-workspaces/workspace-configuration`.

## What already holds

`epic:parallel-workspaces/component-runtime` and the proof commands of `epic:rust-adoption/parity-validation` built part
of this epic's spec, and its slices do not build it again: Root discovery, which finds the nearest ancestor holding the
file `.winter/config.toml`; reading `.winter/state.toml`, with a missing or unparsable file read as an empty registry;
the lookup of Env index allocation; the removal writer; and Write integrity in the file port.

## Consistent invocation snapshot (engineering:configuration-snapshot)

An invocation's target resolution, validation, and effects use a consistent configuration view; a concurrent edit cannot
redirect effects after validation without detection. File formats and snapshot mechanisms remain unselected. Serves
`product:identify-workspace` and `product:configuration-validation`.

## Side-effect-free interpretation (engineering:inert-configuration-read)

Configuration loading and explanation are independent of external-command execution and generated-state writers; failure
paths retain source attribution without logging secret values. Serves `product:effective-configuration`.

## Preserve durable edits (engineering:configuration-write-integrity)

Supported configuration writers preserve unmanaged content and coordinate across processes; interruption cannot turn a
valid previous declaration into an accepted partial declaration. Conflict and recovery policy follows
`decision:parallel-workspaces/workspace-configuration/write-conflicts`. Serves `product:configuration-state-ownership`.
