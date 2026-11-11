# Workspace release requirements

For `epic:rust-adoption/workspace-release`.

## Obtain the complete component (product:complete-workspace-install)

The engineer installs the Workspace CLI, its matching built-in context, and its supported first-party provider behavior
as one usable product. Shared installation rules remain in `product:homebrew-installation`.

Acceptance:

- The released `winter-workspace` component can be invoked directly and through a compatible `winter workspace` launcher
  without Fleet or the old Python CLI.
- Release guidance names the shared CLI baseline and provider-specific support and prerequisites under
  `decision:supported-systems`.
- The component identifies its version, and its framework guidance describes that version rather than a separate copy
  from a workspace fork.

## Qualify a replacement release (product:replacement-release-readiness)

A release presented as the Python Workspace replacement accounts for the inventoried capabilities and supported
first-party providers before recommending cutover. A partial development or preview release is identified as such.

Acceptance:

- Every mapped capability has accepted evidence of its preserved behavior or a product-owner-approved replacement; an
  unimplemented capability or unresolved blocking redesign prevents a rough-parity release claim.
- Each supported first-party provider passes its lifecycle, isolation, readiness, status, log, and teardown acceptance
  without a first-party Python runtime; unsupported target/provider combinations are named before installation.
- The replacement release includes accepted migration and recovery guidance, production platform-handoff evidence, and
  compatibility results rather than relying only on a build or protocol-shape check.
- Preview artifacts state their missing capabilities and cannot be presented as approval to convert a live workspace.

## Understand upgrade cost (product:workspace-upgrade-guidance)

The engineer can learn the configuration, command, extension, and automation changes before upgrading a workspace's
selected component. Changing the installed default does not silently adopt or convert another workspace.

Acceptance:

- Release notes identify breaking differences and the required adoption or extension changes.
- Compatibility guidance distinguishes the platform handoff version, Workspace configuration versions, and extension
  protocol versions; matching marketing versions are not required.
- Pinning or changing the selected component remains a deliberate operation through the agreed platform boundary.
