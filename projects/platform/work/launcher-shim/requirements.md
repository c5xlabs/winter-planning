# Winter launcher shim requirements

For `epic:launcher-shim`.

## Reach an independently installed product (product:reach-component)

A person can use the common entry point for a product without installing every other Winter product.

Acceptance:

- `winter workspace …`, `winter hub …`, and `winter runner …` select the corresponding separately installed role
  executable when it is available through the agreed discovery rules.
- An available component can be used when the other components are absent.
- The platform's own help and version remain platform-owned rather than becoming dispatch targets.
- Hub and runner invocations do not require the current directory to be a Winter workspace.

## Preserve the invocation (product:preserve-invocation)

A person or agent using the shim receives the same command behavior as when invoking the selected component directly.

Acceptance:

- Arguments after the product group reach the component in their original order, including flags and arguments
  containing spaces.
- The Rust component runs in the caller's working directory and receives the caller's input, output, and error streams.
- The component's exit status reaches the caller; an unsuccessful product command is not reported as successful.
- Interrupting or terminating a shim invocation does not leave the component running unnoticed.

## Show help for the invoked entry point (product:invocation-aware-help)

A person receives help and usage guidance that matches the entry point they used, without needing to know the underlying
component executable's name.

Acceptance:

- Help reached through `winter workspace`, `winter hub`, or `winter runner` uses that command prefix in usage lines and
  command examples rather than the underlying executable name or cache path.
- Nested-command help and usage errors preserve the same entry-point prefix.
- Invoking a component directly produces help and usage guidance appropriate to that direct invocation.
- Changing the displayed command prefix does not change the component's identity or reported version.

## Respect the selected version (product:respect-component-version)

A person can work in directories that select different component versions without one changing the other's command,
and can run hub and runner versions without adopting Workspace configuration. Selection follows
`decision:launcher-shim/version-scope`; the pin, inherited-selection, and user-default records are settled by
`decision:launcher-shim/version-selection`.

Acceptance:

- Each workspace's invocation uses its pin in `.winter/config.toml` when present; otherwise it uses the inherited
  `.winterrc` selection for Workspace, then its user-selected default.
- `.winterrc` files can exist in any directory, without a Winter workspace. All files from filesystem root through the
  caller's directory contribute selections, with child values overriding only the roles they specify.
- A child file selecting Runner does not erase the Hub or Workspace selection inherited from an ancestor.
- Hub and Runner use their inherited `.winterrc` selections, then their own user-selected defaults, without reading
  `.winter/config.toml`.
- A home-directory `.winterrc` participates only when it is on the caller's ancestor path, not as a universal fallback.
- Platform-owned `$XDG_CONFIG_HOME/winter/config.toml` records independent user-selected defaults for all three roles;
  an invocation without applicable directory configuration uses that role's default.
- Installing or caching a release does not implicitly select it; selection never guesses the newest cached release or a
  binary on PATH.
- Changing one role's selection does not change another role's selection or override a Workspace configuration pin.
- A selected version that is missing, or a role with no selectable version, produces a clear error rather than silently
  falling back to another version.
- Reusing a cached release does not require a compiler or network access.
- Platform help and version remain usable without resolving component pins or loading workspace configuration.

## Keep existing Python workspaces usable (product:legacy-workspace-dispatch)

An engineer can replace the singleton launcher while existing workspace automation continues to use Python under
`decision:singleton-launcher-succession`, without treating that as Rust Workspace adoption.

Acceptance:

- A defined set of legacy command names launches Python for the discovered target workspace only when its
  `.winter/config.toml` has top-level `enable-legacy-winter = true`; absent or false disables that path even when a
  Python source override is supplied.
- Recognizing the command is sufficient for routing: after consuming the optional first-argument `--winter=PATH`
  launcher control, the remaining legacy argument vector is forwarded unchanged, including leading options, unknown
  options, option values, delimiters, and nested commands, without a Python option schema in the launcher.
- After an opaque Python option prefix begins, values such as `init`, `workspace`, `hub`, and `runner` do not select
  bootstrap or Rust dispatch; the invocation remains flag-gated legacy forwarding under
  `decision:launcher-shim/routing-boundary`.
- `--service-orchestrator` and other Python-owned options are interpreted by Python, not reimplemented in the launcher.
- Explicit `winter workspace`, `winter hub`, and `winter runner` always use Rust components regardless of the legacy
  flag; missing Rust components never trigger Python fallback.
- Legacy dispatch uses neither the Rust cache nor a Workspace release pin and never recursively invokes `winter` from
  PATH or the old installed wrapper.
- Python environment preparation, original-invocation path handling, streams, exit status, and interruption behavior
  remain compatible with the existing launcher; setup output goes only to stderr.
- Missing or disabled legacy support gives an actionable diagnostic instead of invoking Rust under an old spelling.
- Agents can use first-argument `--winter=PATH` to select `<PATH>/tools/winter-cli` for this invocation. Relative paths
  resolve from the original caller cwd, before any launcher chdir.
- The override changes only the Python implementation and its dependency environment, never the target workspace,
  configuration used for legacy opt-in, original invocation directory, Rust pins, or role defaults.
- Selected-source dependency files, venv, and sync stamp control preparation; Python still runs with cwd anchored at the
  target workspace's `tools/winter-cli`.
- Reusing a relocated venv prepared from a different source requires synchronization with the selected source, even when
  the selected source's dependency files are older than the previous synchronization.
- A malformed override or missing/unusable source fails explicitly without falling back to the target's implementation.
  Override-source configuration does not grant legacy permission for the target workspace.
- Launcher-level `--winter=PATH` is rejected for explicit Rust groups rather than interpreted as a Rust source selector;
  Rust development continues to invoke the built sub-executable directly.

## Retain temporary legacy bootstrap (product:legacy-init)

A person can still run `winter init` at a legacy workspace clone's root before configuration exists. This temporary
compatibility path follows `decision:singleton-launcher-succession` and is removed with legacy support.

Acceptance:

- `winter init` requires `tools/winter-cli/` in cwd, not an ancestor, and works without a legacy flag, Rust component,
  directory pin, or user-selected default.
- It creates `.winter/config.toml` only when absent, leaves an existing configuration untouched, and launches that
  target's selected Python entrypoint with `ws init` followed by the supplied arguments.
- It does not silently enable ordinary legacy dispatch; disabled-command diagnostics name the required flag.
- First-argument `--winter=PATH` may select the Python implementation used for bootstrap but never relocates the target
  from cwd or bypasses its `tools/winter-cli/` eligibility check; the source is validated before creating config.
- An empty CLI directory or one without the Python project and entry module is rejected before creating configuration.
- A setup or Python failure reports failure and preserves existing configuration; a newly created configuration may
  remain. Dependency synchronization may update its lockfile and environment.

## Explain platform verbosity (product:launcher-verbosity)

A person can request launcher diagnostics with root `--verbose` or `-v` without changing the selected application or
corrupting its structured output.

Acceptance:

- On modern invocations, root verbosity controls launcher diagnostics on stderr; component verbosity is requested after
  the role group and is passed through unchanged.
- On legacy invocations, original verbosity options are also forwarded to Python with the remaining Python arguments.
- Bare `winter` and root verbosity without a command show successful platform help; `-V` and `--version` identify the
  platform. Root help and version remain configuration-independent even when verbosity is enabled.

## Replace the singleton deliberately (product:singleton-cutover)

An engineer can manually replace the installed wrapper before Rust Workspace parity while choosing each workspace's
adoption separately, under `decision:singleton-launcher-succession` and `decision:launcher-shim/manual-cutover`.

Acceptance:

- Installation and rollback are manual; no cutover documentation, automated installer, or launcher-migration tooling is
  required by this epic.
- Replacing the singleton does not adopt Rust Workspace, convert workspace state, or silently enable legacy dispatch.
- Existing Python agents retain their `winter --winter=PATH …` workflow during the transition; Rust development uses
  direct executable evaluation instead.
- Launcher replacement requires accepted modern and legacy dispatch evidence, not completion of Rust Workspace parity.
  Rust Workspace adoption retains its own capability, provider, and migration gates.
- The legacy adapter and temporary init remain until consumers deliberately migrate to explicit Rust groups; retirement
  follows the suite's release deprecation policy without adding implicit Python fallback or undocumented aliases.

## Reuse cached releases (product:reuse-component-release)

A person can use the same released component version from more than one workspace without keeping a copy in each. The
shared cache follows `decision:version-cache`.

Acceptance:

- The default shared version cache is `~/.cache/winter/versions/`.
- Distinct components and release versions coexist without overwriting one another.
- Selecting a different version for one workspace does not alter the release selected by another.

## Explain dispatch failures (product:explain-dispatch-failure)

A person or agent is told which component or selected version cannot be run, without the shim guessing another command.

Acceptance:

- A missing, incompatible, or non-executable target produces an explanatory error and an unsuccessful exit.
- The error distinguishes a launcher failure from an error returned by a component that actually started.
- An unsupported command is rejected, not forwarded to Python, the existing `winter` command, or an unrelated
  executable.
- Failure does not install components, change the workspace pin, or replace the user's installed command.
