# Executable contribution requirements

For `epic:agent-ready-operation/executable-extensions`.

## Inspect before execution (product:inspect-contributions)

An engineer can discover an extension's declared executable contributions without granting permission to run them.

Acceptance:

- Inspection identifies the extension, executable purpose, source, activation state, and declared compatibility.
- Reading declarations, context, or help does not import a plugin, invoke registration code, run hooks, or run checks.
- A missing, disabled, or unreadable contribution is distinguished from a contribution that ran and returned no results.

## Reject incompatible contributions (product:compatible-contributions)

An engineer learns that a provider is incompatible before Workspace asks it to act. Automatic selection does not weaken
the checks applied to an explicit choice.

Acceptance:

- Explicit and implicit selections apply the same compatibility, entrypoint, and eligibility rules to every provider.
- An incompatible or unusable member of a selected provider set is named before action dispatch begins.
- Inspection shows the supported contract versions and the reason a declared version cannot be used.
- Undeclared legacy versions receive an explicit compatibility policy, not accidental acceptance through a different
  selection path; its choice is blocked by `decision:agent-ready-operation/executable-extensions/protocol-integration`.

## Complete or stop external work (product:controlled-execution)

An unattended agent can tell whether an external contribution completed or failed, and does not mistake executable
chatter for a Workspace result. Workspace runs external work as the reference tool does, with no time or output limit of
its own except on an environment lookup command, under
`decision:agent-ready-operation/executable-extensions/execution-policy`.

Acceptance:

- Launch failure, refusal, unsuccessful exit, invalid response, timeout, and cancellation are distinguishable outcomes.
- A failing contribution does not hide unrelated target outcomes or prevent supported inspection and recovery commands.
- Execution identifies the effective workspace, extension, scope, writable asset location, and interaction policy.
- Cancellation and the one execution limit report unfinished work and known partial effects; they do not claim to roll
  back an external program's changes.
- External work other than an environment lookup command has no execution or output limit, and a timeout is an outcome
  of that command only. Interaction policy is explicit under
  `decision:agent-ready-operation/executable-extensions/execution-policy`.

## Observe lifecycle contributions (product:lifecycle-contributions)

An engineer can rely on hooks running at declared workspace and environment phases, including when setup or teardown
fails. The failure policy is visible before destructive work begins.

Acceptance:

- Workspace reconciliation runs its workspace hook once per requested workspace reconciliation, before any subsequent
  environment loop; environment-only reconciliation does not silently run that workspace hook.
- Environment initialization exposes the assigned environment identity and reports whether structural prerequisites
  completed before its hook is attempted or skipped.
- Environment destruction attempts its applicable hooks while the target worktrees still exist; strict failure stops
  their removal, and non-strict continuation reports the failure and remaining effects.
- Hook order, repeat-run behavior, and sibling-failure continuation are declared rather than inferred from dependency
  names; `decision:agent-ready-operation/executable-extensions/hook-policy` owns the unresolved choices.
- Hooks do not silently replace the provisioning phases owned by `epic:running-environments/resource-provisioning`.

## Check an extension's conformance (product:safe-conformance)

An extension author can create a starting extension and assess its declared compatibility. The check runs the
extension's real program in the live workspace, with no isolation, as the reference tool does, under
`decision:agent-ready-operation/executable-extensions/conformance-policy`.

Acceptance:

- Scaffolding refuses to overwrite existing user content without explicit authorization and identifies generated files.
- The starting extension explains its supported and refused operations; recognition is not presented as implemented
  service behavior.
- A conformance check invokes the extension's real entrypoint against the workspace it is run in; nothing stands in for
  the entrypoint or the workspace.
- Results distinguish contract recognition, output validity, compatibility, and behavior not established by the check.
- An author's external program may choose its runtime, but a generated example does not impose Python on first-party
  Workspace behavior.
