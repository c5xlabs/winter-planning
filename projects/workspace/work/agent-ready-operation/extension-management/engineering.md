# Extension management engineering requirements

For `epic:agent-ready-operation/extension-management`.

## What already holds

`engineering:domain-io-boundaries` separates declarations from effects, and `engineering:concurrent-state-safety`
protects authoritative changes. Configuration merging belongs to `epic:parallel-workspaces/workspace-configuration`;
repository refs and locks belong to `epic:parallel-workspaces/reproducible-inputs`, not a second extension installation
database.

## Explicit contribution provenance (engineering:contribution-provenance)

Resolved declarations carry repository kind, stable owner, source, configured overrides, and effective eligibility
explicitly; consumers cannot infer project-versus-standalone identity from a path beneath `projects/`. Activation,
inspection, and dispatch evaluate the same rules for a configuration snapshot. Serves `product:extension-inventory` and
`product:extension-activation`.

## Manifest validation boundary (engineering:manifest-validation)

The declaration contract distinguishes absence, invalid values, unsupported fields or versions, and deliberate opt-out.
Path-bearing declarations and namespace components are validated before projection or execution, including resolved
symlink escapes; schema evolution and error-isolation choices remain blocked by
`decision:agent-ready-operation/extension-management/manifest-policy`. Serves `product:extension-inventory` and
`product:declared-contributions`.

## Reconciliation authority (engineering:activation-transitions)

Activation reconciles from known prior and desired contribution ownership, not current filename guesses or a malformed
manifest's defaults. An incomplete transition remains diagnosable, and simultaneous same-workspace reconciliation must
serialize or refuse conflicting updates across processes under the selected ownership policy. Serves
`product:extension-reconciliation`.

## Exclude containment (engineering:exclude-authority)

Declared excludes reject negation, traversal, filesystem-root forms, control characters, and workspace-root matches.
Artifact-kind paths are normalized and literal path metacharacters escaped. Managed edits preserve text outside owned
blocks and refuse ambiguous malformed markers rather than consuming user text through end-of-file. Serves
`product:extension-assets`.
