# Artifact space requirements

For `epic:agent-ready-operation/artifact-space`.

## Resolve an artifact destination (product:artifact-destination)

A tool can ask for the absolute destination of an artifact kind without preparing the workspace to write there.

Acceptance:

- An unconfigured kind resolves below the workspace's default artifact root, with no fixed catalog of permitted kinds.
- The answer is usable directly by a non-interactive caller without stripping explanatory text.
- Resolution creates no directory, writes no file, invokes no contribution, and makes no repository change.
- A malformed kind or unusable configuration returns an unsuccessful result without printing a usable destination.

## Configure shared and personal locations (product:artifact-location-overrides)

An engineer can set a common root and redirect individual kinds without changing every consuming tool.

Acceptance:

- The default root is the workspace's `.winter` directory; a relative configured root is workspace-relative.
- A relative kind override is relative to the effective root; home-relative and absolute root or kind overrides can
  intentionally locate artifacts outside `.winter` and outside the workspace.
- The normal local configuration overlay applies to root and kind choices.
- A well-formed unknown kind still resolves using its name; resolving does not require the destination to exist.

## Keep kinds distinct from paths (product:artifact-kind-safety)

A caller names one artifact bucket, not a path that can select an unrelated directory.

Acceptance:

- A kind is a single name beginning with an ASCII letter or digit and continuing with letters, digits, dots,
  underscores, or hyphens; empty names, separators, and traversal operands are refused.
- Rejecting an unsafe operand does not forbid trusted configured absolute or home-relative destinations.

## Separate finding a path from maintaining it (product:artifact-materialization-ownership)

An engineer can resolve a destination without assuming that Workspace owns existing files there.

Acceptance:

- Consumers decide when to create and write their destination and how generated content is tracked or excluded.
- Resolving a kind neither cleans its contents nor registers the directory for later deletion.
- Any declared cleanup remains governed by the ownership and safeguards of
  `epic:running-environments/resource-provisioning`, including destinations outside the workspace.
