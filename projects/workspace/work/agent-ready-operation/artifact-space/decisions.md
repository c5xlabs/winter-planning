# Artifact space decisions

## Path normalization (decision:agent-ready-operation/artifact-space/path-normalization)

**Decider:** product owner and engineering

**Open.** Choose lexical path resolution or filesystem-aware canonicalization for configured destinations, preserving
nonexistent destinations and intentional external roots either way; this blocks finalizing normalization behavior in
`product:artifact-location-overrides`, not the pure-resolution and no-cleanup promises.
