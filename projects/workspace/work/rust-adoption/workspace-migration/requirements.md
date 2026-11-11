# Existing-workspace adoption requirements

For `epic:rust-adoption/workspace-migration`.

## Inspect before adopting (product:adoption-preview)

The engineer can learn whether their workspace can be adopted before any files or resources change. The accepted
conversion boundary is chosen in `decision:configuration-transition`.

Acceptance:

- Inspection distinguishes supported declarations, convertible legacy fields, unsupported extensions, and
  workspace-specific customizations that need a person's judgment.
- The preview names the files, version selection, and generated projections adoption would change.
- Existing worktrees and repositories remain usable after inspection, even if adoption is refused.

## Keep the user's workspace (product:owned-content-survives)

Adoption preserves the engineer's declarations, local overlay, project context, feature branches, and data. Framework
files are removed or replaced only when their ownership and replacement are known.

Acceptance:

- User edits and files with ambiguous ownership stop automatic replacement and identify the conflict.
- Existing environment identities, repository locations, and resource assignments are preserved or require an explicit,
  reviewed remapping before use.
- A stopped or failed adoption explains what changed and how to recover without deleting feature work.
- Repeating a completed adoption does not duplicate hooks, extension declarations, or agent projections.

## Switch tools deliberately (product:explicit-cutover)

The engineer chooses when the Rust component becomes their Workspace tool and understands changes to commands,
configuration, and extensions before doing so.

Acceptance:

- Guidance maps existing capability names to their replacement entry points and lists deliberate incompatibilities.
- Live replacement cutover requires accepted capability, supported-provider, migration/recovery, and production-handoff
  evidence with no unresolved blocking redesign; source-build experiments and read-only previews are not that approval.
- Before that gate is met, evaluation is limited to explicitly disposable targets rather than adopting a live workspace;
  the gate consumes acceptance evidence and does not depend on completion of the release epic.
- Installation and adoption explain the existing Python `winter` command's PATH/linking conflicts rather than replacing
  or invoking it silently.
- Recovery guidance identifies which changes can be reversed and which state changes require a backup or forward repair;
  it does not promise that an older CLI can read a newly converted workspace.

## Keep maintained integrations usable (product:consumer-transition)

An engineer using maintained workflow skills, conventions, editor integration, or published guidance can drive the
replacement without being sent back to retired Python commands or obsolete output contracts.

Acceptance:

- The supported-consumer inventory includes maintained skills and agents, `winter-context`, `winter-nvim`, and the docs
  site where they refer to Workspace commands, paths, configuration, or structured results.
- Replacement guidance and supported integrations agree on the chosen commands and context entry points before live
  cutover is recommended; an incompatible consumer is identified with its required update or explicit support decision.
- Consumer changes are coordinated with the relevant capability contracts rather than papered over by a Python fallback
  or undocumented compatibility alias.
