# Artifact space engineering requirements

For `epic:agent-ready-operation/artifact-space`.

## Resolution has no write authority (engineering:artifact-resolution-purity)

Destination resolution depends only on effective path configuration, the workspace location, and home expansion; it does
not require destination existence or writable access, create metadata, or allocate cleanup ownership. Serves
`product:artifact-destination` and `product:artifact-materialization-ownership`.

## Validate the operand before joining (engineering:artifact-operand-containment)

The kind operand must satisfy the entire single-name restriction before any path joining; control characters and
trailing unvalidated input are rejected. Trusted path configuration is evaluated separately rather than forced through
the kind restriction. Serves `product:artifact-kind-safety`.

## No ambient destination discovery (engineering:artifact-path-determinism)

Relative path evaluation uses the selected workspace and effective root, not process cwd or a harness-specific home
directory. Home expansion cannot execute a command, and any normalization choice follows
`decision:agent-ready-operation/artifact-space/path-normalization`. Serves `product:artifact-location-overrides`.
