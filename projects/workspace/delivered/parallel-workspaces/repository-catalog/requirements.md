# Repository catalog requirements

For `epic:parallel-workspaces/repository-catalog`.

## Distinguish repository roles (product:repository-roles)

An engineer can declare project repositories that participate in feature environments and standalone repositories held
once per workspace, and the inventory tells the two apart from each other and from the repositories the workspace finds
on its own.

Acceptance:

- The repository list shows each repository's role, effective name, and location, and whether a project is pinned; the
  human list also shows its source.
- Pinning applies only to a project and a branch, tag, or commit reference only to a standalone; adding a repository
  with the other role's setting is refused.
- The list marks the implicit `product` and `harness` repositories as implicit, and the location listing marks the
  workspace itself.
- One repository may be declared both as a project and as a standalone, following
  `decision:parallel-workspaces/repository-catalog/repository-identities`.

## Choose names and locations deliberately (product:repository-addresses)

An engineer can use URL-derived names or explicit aliases, per-repository main branches, and relative standalone
locations without two declarations of one role silently claiming the same name, source, or checkout.

Acceptance:

- Supported URL forms derive a predictable name, and an explicit alias takes precedence.
- A per-repository main branch is recorded with the declaration and overrides the workspace default.
- A source that is not a recognised Git URL, and a source or name already declared for the same role, are refused before
  the inventory changes.
- A standalone location given as an absolute path, with a `..` segment, or resolving to the location of a declared
  standalone is refused, following `decision:parallel-workspaces/repository-catalog/location-policy`.

## Edit the intended inventory layer (product:edit-repository-inventory)

The engineer can add or remove a repository declaration in shared or local configuration while retaining unrelated
settings and, as far as `decision:toml-parity` keeps them, comments.

Acceptance:

- An edit goes to the shared file unless the engineer asks for the local one, and reports the role and the source or
  name it changed; an unknown removal is refused rather than reported as success.
- Adding a repository refuses a source or name already declared for the same role in either layer; hand-authored
  duplicates follow the configuration rules of `epic:parallel-workspaces/workspace-configuration`.
- A layer whose repositories cannot be edited safely is refused unchanged, following
  `decision:parallel-workspaces/repository-catalog/refused-edits`.
- Removing a declaration does not delete its checkout, worktrees, branches, or provisioned resources.
- Adding a declaration does not implicitly clone, fetch, run setup commands, or execute extension content.
- Concurrent edits either retain both non-conflicting changes or explain the conflict.

## See declared versus present repositories (product:inventory-presence)

An engineer, or an editor integration offering places to jump to, can enumerate the checkout locations that exist and
learn when declared projects and the `projects/` directory have drifted apart.

Acceptance:

- The repository list warns when a declared project has no checkout under `projects/` and when a directory there is not
  declared.
- The location listing offers concrete paths for existing feature worktrees, standalones, and the workspace itself, and
  leaves out locations that do not exist.
- Neither listing creates a location or runs a repository's setup commands.
- Optional Git details give ahead, behind, and dirty counts for feature worktrees and the workspace, give none for a
  standalone or for a workspace whose probe fails or whose checkout is detached, and stop the listing with the failure
  when a worktree's probe fails.
