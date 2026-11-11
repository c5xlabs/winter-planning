# Harness projection requirements

For `epic:agent-ready-operation/harness-projection`.

## Protect canonical and local work (product:projection-ownership)

An engineer can reconcile generated harness files without risking canonical extension sources or their hand-authored
harness configuration.

Acceptance:

- Projection distinguishes owned generated artifacts, modified owned artifacts, user destination conflicts, obsolete
  artifacts, and ambiguous ownership before updating or deleting them.
- A matching prefix or filename alone does not authorize replacement or cleanup; equal and dash-overlapping namespaces
  are checked across workspace skills, standalone extensions, and project extensions before writes.
- Canonical sources and unrelated files remain unchanged, including when replacing a legacy symlink to a canonical agent
  with a rendered copy.
- Source, prefix, enablement, and extension-set changes expose conflicts and retained artifacts instead of guessing
  ownership or deleting another live extension's files.
- Unknown or corrupt ownership information refuses destructive cleanup and explains recovery under
  `decision:agent-ready-operation/harness-projection/artifact-ownership`.

## Discover skills on supported harnesses (product:skill-discovery)

A person can use workspace-authored and extension skills through their preferred supported harness without manually
copying skill assets.

Acceptance:

- Claude Code discovers projected skills under `.claude/skills`, Codex under `.codex/skills`, and OpenCode under
  `.opencode/skill`; symlink/copy behavior respects each supported harness's discovery limits.
- A source skill is a child directory containing `SKILL.md`, with its assets retained; default source discovery tries
  `skills` before `.claude/skills`, while an explicit directory replaces fallback.
- Installed names are `<prefix>-<directory>`, except a directory equal to its prefix installs as the bare prefix;
  workspace skills default to namespace `ws` and source `skills` independently of extension adoption.
- Canonical skill names cannot silently defeat namespace naming; OpenCode's copied frontmatter carries the installed
  name while canonical source remains unchanged.
- Unchanged projections are not rewritten, changed source content becomes current, and removed skills trigger only
  authorized cleanup; copied assets are not mistaken for a second canonical source.

## Discover agents on supported harnesses (product:agent-discovery)

An engineer can install one canonical agent into each harness's native format and identify the name callers must use.

Acceptance:

- Claude artifacts are Markdown under `.claude/agents`, Codex artifacts are TOML under `.codex/agents`, and OpenCode
  artifacts are Markdown under `.opencode/agent`, with prefixed filenames.
- Claude/Codex expose canonical agent identity; OpenCode invocation identity follows its prefixed filename. Inspection
  distinguishes both identities rather than promising one unqualified invocation name everywhere.
- Agent bodies reach Claude/OpenCode as prompt text and Codex as `developer_instructions`, without copying another
  harness's override block or silently altering the reusable procedure.
- OpenCode agents default to subagent mode unless an accepted native override changes it; model and effort use
  `epic:agent-ready-operation/agent-model-resolution` rather than a projection-specific resolver.
- Loss of common fields, including tool restrictions, follows the explicit permission-loss policy rather than claiming
  that native sandbox and permission settings are equivalent.

## Deliver the correct context entry point (product:harness-context-entry)

An agent receives the workspace's essential safety guidance and a route to the relevant context without replacing
project-owned instructions.

Acceptance:

- Harness entry points expose the context selected by `epic:agent-ready-operation/context-navigation`, including eager
  safety text when the harness cannot execute session-start commands.
- Eager versus lazy delivery and project worktree routing preserve the selected source and loading intent; a project
  source checkout is not injected as the active feature copy.
- Updating generated context leaves authored `AGENTS.md`, the supported Claude shim, optional local instructions, and
  unrelated harness configuration intact.
- Removing the final applicable extension updates or removes its generated routing content, without leaving a stale
  active declaration or deleting unrelated context.
- Legacy generated shims are removed only under explicit migration ownership, not merely because their names resemble an
  old Winter file.

## Recover from incomplete projection (product:projection-recovery)

An engineer can tell what remains installed when a source becomes invalid or projection stops halfway through.

Acceptance:

- Every requested harness render for an agent is validated before any of that agent's copies are changed; a rendering
  failure cannot silently update only the earlier harnesses.
- An unavailable source, invalid model mapping, permission refusal, copy failure, interruption, and source removal are
  distinct conditions; none is silently treated as permission to prune all prior good output.
- Retaining a previous good copy is reported as retained/stale rather than current; withdrawal is reported explicitly
  under `decision:agent-ready-operation/harness-projection/last-good-policy`.
- A filesystem failure reports the actual per-harness changes and recoverable remainder rather than claiming an atomic
  all-harness transaction; repeating reconciliation can repair owned incomplete output safely.
- Competing reconciliations do not overwrite each other's accepted ownership updates or report conflicting artifacts as
  current; the cross-process policy is finalized with
  `decision:agent-ready-operation/harness-projection/write-recovery`.

## Inspect projection health without repair (product:projection-drift)

A person or agent can find stale, missing, conflicted, and orphaned harness artifacts before requesting repair.

Acceptance:

- Inspection compares the expected transformed content and destination with current installed artifacts, including
  bare-prefix skills, symlink targets, copied skill assets, and agent model/effort changes.
- Resolution errors and unreadable sources are reported as causes, not mislabeled as ordinary orphaned output.
- Unmanaged local files are not incorrectly claimed as orphans; unavailable ownership is shown explicitly.
- Inspection never repairs artifacts, runs hooks, or rewrites configuration, and exposes results to
  `epic:agent-ready-operation/workspace-diagnostics` and agent-setting inspection.
