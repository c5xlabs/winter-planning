# Workspace component requirements

For `epic:parallel-workspaces/component-runtime`.

## Inspect the component before setup (product:component-help-version)

A person can discover Workspace commands and identify the installed component before creating a workspace. Help
distinguishes implemented capabilities from the migration plan.

Acceptance:

- Help and version work outside a workspace without Fleet, Python, configuration, or provider processes.
- Direct invocation identifies the Workspace component, not the platform version.
- Help and usage guidance use the entry point selected by the caller, including nested commands, under
  `product:platform/launcher-shim/invocation-aware-help`.
- Unsupported commands are refused rather than forwarded to a legacy executable.

## Drive commands unattended (product:automation-boundary)

An agent can select a target and distinguish success, refusal, usage error, and partial completion without reading a
terminal screen. Command vocabulary and compatibility are settled by `decision:command-vocabulary`.

Acceptance:

- Non-interactive commands never stop at an undisclosed prompt; missing approval or input is reported as an actionable
  refusal.
- Supported structured modes describe the command result separately from progress and diagnostics.
- Invalid operands and unknown commands report usage without starting the requested mutation.
- Calling the selected component through the platform preserves its input, output, exit, and interruption behavior.

## Learn the replacement vocabulary (product:discover-command-scope)

The engineer can distinguish creating a new workspace from reconciling an existing declaration, and discover which
commands require a workspace or environment.

Acceptance:

- Creation refuses an already initialized target instead of treating it as a repair operation.
- Reconciliation describes the existing scope it converges and is safe to repeat under the capability's safety rules.
- Help names destructive effects and required scope rather than relying on an old command spelling to imply them.
- Final names and flags are chosen together before public contracts freeze; illustrative strategy commands are not
  advertised as already supported.

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
