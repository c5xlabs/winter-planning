# Workspace component requirements

For `epic:parallel-workspaces/component-runtime`.

## Inspect the component before setup (product:component-help-version)

A person can discover Workspace commands and identify the installed component before creating a workspace. Help
distinguishes implemented capabilities from the migration plan.

Acceptance:

- Help and version work outside a workspace without Fleet, Python, configuration, or provider processes.
- Direct invocation identifies the Workspace component, not the platform version.
- Usage, help, and the version line name the component `winter-workspace` whatever name it was invoked under, under
  `decision:parallel-workspaces/component-runtime/program-name`.
- Unsupported commands are refused rather than forwarded to a legacy executable.

## Drive commands unattended (product:automation-boundary)

An agent can select a target and distinguish success, refusal, usage error, and partial completion without reading a
terminal screen. Command vocabulary and compatibility are settled by `decision:command-vocabulary`.

Acceptance:

- Every confirmation prompt is skipped by an explicit force or dry-run option; a prompt that cannot read an answer ends
  as a failure before the mutation starts. Structured mode alone suppresses no prompt, except when pruning orphans,
  where it removes the safe ones without asking.
- Supported structured modes describe the command result separately from progress and diagnostics, except for the
  plain-text lines the reference tool writes in structured mode, which are kept under
  `decision:parallel-workspaces/component-runtime/json-mode-purity`.
- Errors are plain text on the diagnostic stream in every mode; an agent decides success from the exit code, under
  `decision:parallel-workspaces/component-runtime/error-rendering`.
- Invalid operands and unknown commands report usage without starting the requested mutation.

## Learn the command scope (product:discover-command-scope)

The engineer can discover which commands require a workspace or an environment, and what each one converges or destroys.
Names and flags are the reference tool's, under `decision:command-vocabulary`.

Acceptance:

- Reconciliation describes the existing scope it converges and is safe to repeat under the capability's safety rules.
- Help names destructive effects and required scope rather than relying on a command spelling to imply them.
- Help lists only commands the binary implements; the launcher's workspace-creation command is not among them, under
  `decision:parallel-workspaces/component-runtime/launcher-only-surface`.

## Evaluate a modified component (product:local-component-evaluation)

An engineer or agent changing Workspace can invoke their locally built component directly against an intended workspace
without replacing the installed tool or changing that workspace's release selection. The workflow follows
`decision:development-component-selection`.

Acceptance:

- A local build is evaluated by invoking its `winter-workspace` executable directly, not by replacing a release or
  selecting a feature environment through the installed launcher.
- Agent/runtime guidance in the `winter-workspace` code repository explains how to build and invoke that executable
  from a feature-environment worktree and is discoverable through the repository's agent entry point.
- Choosing different code does not change or sandbox the target workspace; guidance states the actual mutation scope.
- Evaluation leaves the installed component, other workspaces' pins, and immutable released cache entries unchanged.
- Guidance states that Rust source changes require rebuilding before the next invocation and distinguishes direct Rust
  execution from the Python-only `--winter=PATH` compatibility path under
  `decision:platform/singleton-launcher-succession`; no Rust launcher override or automatic rebuild is required.
