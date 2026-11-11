# Extension management requirements

For `epic:agent-ready-operation/extension-management`.

## Identify installed contributions (product:extension-inventory)

An engineer can tell which repository and declaration supply an extension, independently of its local install prefix.

Acceptance:

- Inventory distinguishes repository identity, manifest identity, source checkout, prefix, and contribution kinds;
  conflicting identities are reported rather than silently substituted.
- Standalone extensions and project-repository extensions are discoverable; a project extension's contribution source is
  its designated source checkout, not an arbitrary feature worktree.
- When the same repository is declared as standalone and project, the selected contribution source is deterministic and
  visible; removing one declaration does not create duplicate active extensions.
- Manifest declarations cover context load, skill/agent roots, hooks, checks, dependencies, capabilities and versions,
  excludes, and contributed service/provision declarations without running those contributions.
- Malformed declarations identify the extension, field, and affected contribution rather than appearing empty and valid.

## Separate acquisition and activation (product:extension-activation)

An engineer can keep a repository installed while choosing whether it supplies agent or executable capabilities.
`decision:agent-ready-operation/extension-management/adoption-policy` owns the adoption policy and defaults.

Acceptance:

- Repositories opted out of extension behavior remain available for repository operations and selected-ref management.
- Manifest-based adoption, explicit adoption of a manifest-less standalone, context-only adoption, and extension opt-out
  are distinguishable choices; permissive adoption does not silently activate unrelated contribution kinds.
- Project repositories without a manifest can supply explicitly requested context without thereby supplying hooks,
  checks, skills, agents, or capability providers.
- Nested workspaces remain outside the outer workspace's extension activation, consistent with
  `epic:parallel-workspaces/nested-workspaces`.
- The effective policy applies consistently to every consumer; inspection explains disabled or ineligible contributions.
- Installation or revision changes do not implicitly authorize lifecycle execution; acquisition/ref safety remains with
  `epic:parallel-workspaces/repository-catalog` and `epic:parallel-workspaces/reproducible-inputs`.

## Reconcile a changed extension set (product:extension-reconciliation)

An engineer can repeat activation after changing declarations without accumulating unexplained active contributions or
losing unrelated work.

Acceptance:

- Repeating unchanged reconciliation leaves the effective contribution set unchanged and reports no fictitious updates.
- Add, disable, remove, source relocation, and prefix changes identify the contributions added, retained, withdrawn,
  conflicted, or not attempted; removing the last extension is handled explicitly.
- Disabling context alone does not disable other contributions; disabling an extension does not delete its repository,
  writable assets, or external resources without separately authorized work.
- Unreadable or missing declarations do not authorize guessed cleanup or silently increase eager context loading.
- Unrelated authored configuration and instructions survive reconciliation; artifact cleanup follows the ownership
  policy of `epic:agent-ready-operation/harness-projection`.

## Supply declarations to their owners (product:declared-contributions)

An engineer uses extension-declared checks, context, dependencies, and runtime contributions through the surfaces used
for workspace-owned declarations.

Acceptance:

- Context delivery selects the first existing configured entry candidate, reports an explicitly requested missing entry,
  and distinguishes eager, lazy, and suppressed delivery through `epic:agent-ready-operation/context-navigation`.
- A project extension does not eagerly inject the source checkout as if it were the agent's active feature worktree.
- Dependency declarations are visible to `epic:agent-ready-operation/dependency-graph` and checkout-scoped quality
  checks; declaring a dependency does not automatically acquire or execute it.
- Doctor/lint scripts, capability declarations, and service/provision declarations reach their owning command families
  with the same effective identity and eligibility used by management inspection.
- Workspace-authored skills remain a distinct source and namespace, even when extension adoption is disabled.

## Keep extension assets local (product:extension-assets)

An engineer can configure an installed extension without editing its source or accidentally committing machine-local
assets.

Acceptance:

- The resolved writable configuration/asset location is visible and passed consistently to executable contributions; it
  is distinct from the extension source location and honors supported workspace overrides.
- Shared settings, personal overlays, generated artifacts, and machine-local state remain distinguishable.
- Extension-declared workspace excludes are validated and applied as a whole; an invalid declaration does not hide the
  workspace root or partially install exclusion rules.
- Excludes preserve unrelated rules and tracked ignore files; removing declarations removes only their owned rules.
- Artifact-kind excludes follow `epic:agent-ready-operation/artifact-space`, without creating directories during lookup
  or applying workspace ignores to locations outside the workspace.
