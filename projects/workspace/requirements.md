# Workspace product requirements

For Winter Workspace.

## Explain command results (product:explain-results)

A person or coding agent can distinguish completed work, refused work, and work that stopped after changing only part of
the target. They do not have to infer success from the absence of a traceback.

Acceptance:

- Commands that act on several repositories, environments, extensions, or resources identify affected targets and their
  individual outcomes, including targets skipped or not attempted.
- A refusal names the unsafe or unmet condition and the target; a partial failure states what changed and what remains.
- Commands fail unsuccessfully when their promised work did not complete; a successful child action does not hide an
  unsuccessful overall result.
- Supported machine-readable modes can be consumed without stripping progress text, color codes, or diagnostic prose.

## Preserve unrelated work (product:preserve-unrelated-work)

A person can let an agent manage one environment without risking another environment or their own files. Destructive
operations expose their scope and refuse unsafe work rather than assuming permission to discard it.

Acceptance:

- Read-only inspection does not create repositories, allocate resources, run setup hooks, or rewrite configuration.
- Uncommitted or unpushed work, occupied resources, and user-owned files are protected according to the operation's
  stated safety rules; refusal explains how to proceed deliberately.
- A command scoped to one environment does not destroy another environment's worktrees, data, or running services.
- An override, where supported, is explicit about the safeguard it relaxes and does not silently expand the target.

## Use Workspace without Fleet (product:independent-workspace)

An engineer can manage a local workspace without a Fleet hub, runner, or account. Agents and optional Fleet automation
drive the same public Workspace behavior as the person.

Acceptance:

- Workspace setup, repository operations, provisioning, service orchestration, and inspection do not require Fleet.
- Automations can select explicit targets and request structured results without answering interactive prompts.
- Workspace never requires a running Workspace daemon to perform its commands.

## Select targets by pattern (product:target-patterns)

An engineer or agent can name the environments and repositories a command acts on with one pattern language across
Workspace, and a typo never widens what the command touches.

Acceptance:

- A target pattern is `<env>/<repo>`, split at its first `/`; a bare `<env>` means `<env>/*`. Each half matches
  case-sensitively, and a half is a glob only when it holds `*`, `?`, or `[`; otherwise it is an exact name. `**` means
  the same as `*`.
- Commands that default to every worktree when given no pattern, and commands that require one, keep the reference
  tool's split; a command never chooses its target from the working directory.
- A literal environment that is neither registered nor present on disk is refused before the command has any effect.
- A pattern that matches nothing changes nothing and says so, with the result each command's spec states.
- A selected worktree missing from disk is reported and skipped, never treated as a clean or empty repository.
