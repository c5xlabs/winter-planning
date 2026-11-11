# Repository catalog requirements

For `epic:parallel-workspaces/repository-catalog`.

## Distinguish repository roles (product:repository-roles)

An engineer can declare project repositories that participate in feature environments and standalone repositories held
once per workspace, without either role automatically opting into executable extension behavior.

Acceptance:

- Inventory identifies each declared repository's role, effective name, source, intended location, and main branch.
- Project main-branch pinning is distinguishable from a standalone branch, tag, or commit reference.
- Pinned projects remain available in environments while their feature-tracking participation is explicit.
- The workspace repository and any supported implicit repositories are distinguishable from user declarations.
- Dual project/standalone declarations and extension eligibility follow
  `decision:parallel-workspaces/repository-catalog/repository-identities`, not accidental list order.

## Choose names and locations deliberately (product:repository-addresses)

An engineer can use URL-derived names or explicit aliases, mixed main-branch names, and supported relative standalone
locations without two declarations silently claiming the same checkout.

Acceptance:

- Supported URL forms derive a predictable name, and an explicit alias takes precedence.
- Per-repository main branches override the workspace default and are visible before bulk Git operations.
- Invalid names, conflicting identities, overlapping managed locations, and unsafe locations are diagnosed before use.
- Absolute paths, traversal, reserved workspace areas, and symlink escapes cannot silently redirect managed writes.
- Details of intentional shared locations follow `decision:parallel-workspaces/repository-catalog/location-policy`.

## Edit the intended inventory layer (product:edit-repository-inventory)

The engineer can add or remove a repository declaration in shared or local configuration while retaining unrelated
settings and comments.

Acceptance:

- An edit identifies the declaration and configuration layer changed; an unknown removal is not reported as success.
- Duplicate names, sources, and locations are handled consistently for edited and hand-authored declarations.
- Removing a declaration does not delete its checkout, worktrees, branches, or provisioned resources.
- Adding a declaration does not implicitly clone, fetch, run setup commands, or execute extension content.
- Concurrent edits either retain both non-conflicting changes or explain the conflict.

## See declared versus present repositories (product:inventory-presence)

An engineer or editor can enumerate declared repositories and existing checkout locations without mistaking an absent
checkout for an empty clean repository.

Acceptance:

- Declared, present, missing, and undeclared repositories are distinguishable, including standalone locations.
- Location enumeration offers concrete paths for existing worktrees, standalones, and the workspace itself.
- Enumeration does not create missing locations or run repository commands declared for setup.
- Optional Git details identify unavailable or failed probes instead of reporting fabricated clean status.
