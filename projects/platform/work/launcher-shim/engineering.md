# Winter launcher shim engineering requirements

For `epic:launcher-shim`.

## Controlled dispatch verification (engineering:fixture-only-dispatch)

Dispatch verification uses isolated fixture executables and fixture configuration/cache directories; it never invokes
the installed `winter`, the current Python Workspace CLI, or live Workspace/Fleet components. Serves
`product:reach-component`, `product:respect-component-version`, `product:preserve-invocation`, and
`product:invocation-aware-help` under `decision:launcher-shim/component-boundary`.

Acceptance:

- Fixture components expose the arguments, working directory, streams, exit status, and signals received for assertion.
- Fixture-based tests cover shim and direct entry points and assert the command prefix in help, nested-command help, and
  usage errors without changing component identity or version.
- Version-selection cases use isolated cache entries and prove that an unavailable selection does not start a fallback.
- Version-selection cases give the workspace, hub, and runner roles different default versions and assert that each
  role, invoked inside and outside a pinned workspace, reaches its own selection.
- Verification does not read or modify the user's real version cache, workspace pins, or role defaults.
