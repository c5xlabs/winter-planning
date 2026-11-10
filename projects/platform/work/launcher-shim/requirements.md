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
- The component runs in the caller's working directory and receives the caller's input, output, and error streams.
- The component's exit status reaches the caller; an unsuccessful product command is not reported as successful.
- Interrupting or terminating a shim invocation does not leave the component running unnoticed.

## Respect the selected version (product:respect-component-version)

A person can work in two workspaces that select different component versions without one changing the other's command.
The pin and unpinned-selection rules are settled by `decision:launcher-shim/version-selection`.

Acceptance:

- Each workspace's invocation reaches the component version selected for that workspace.
- A selected version that is missing produces a clear error rather than silently falling back to another version.
- Reusing a cached release does not require a compiler or network access.
- Platform help and version remain usable without resolving component pins or loading workspace configuration.

## Reuse cached releases (product:reuse-component-release)

A person can use the same released component version from more than one workspace without keeping a copy in each.
The shared cache follows `decision:version-cache`.

Acceptance:

- The default shared version cache is `~/.cache/winter/versions/`.
- Distinct components and release versions coexist without overwriting one another.
- Selecting a different version for one workspace does not alter the release selected by another.

## Explain dispatch failures (product:explain-dispatch-failure)

A person or agent is told which component or selected version cannot be run, without the shim guessing another command.

Acceptance:

- A missing, incompatible, or non-executable target produces an explanatory error and an unsuccessful exit.
- The error distinguishes a launcher failure from an error returned by a component that actually started.
- An unsupported group is rejected, not forwarded to the existing `winter` command or to an unrelated executable.
- Failure does not install components, change the workspace pin, or replace the user's installed command.
