# Winter launcher shim engineering requirements

For `epic:launcher-shim`.

## Controlled dispatch verification (engineering:fixture-only-dispatch)

Dispatch verification uses isolated fixture executables and fixture configuration/cache directories; it never invokes
the installed `winter`, the current Python Workspace CLI, or live Workspace/Fleet components. Serves
`product:reach-component`, `product:respect-component-version`, `product:preserve-invocation`, and
`product:invocation-aware-help` as well as `product:legacy-workspace-dispatch`, `product:legacy-init`,
`product:launcher-verbosity`, and `product:singleton-cutover` under `decision:launcher-shim/component-boundary`.

Acceptance:

- Fixture components expose the arguments, working directory, streams, exit status, and signals received for assertion.
- Fixture-based tests cover shim and direct entry points and assert the command prefix in help, nested-command help, and
  usage errors without changing component identity or version.
- Version-selection cases use isolated cache entries and prove that an unavailable selection does not start a fallback.
- Version-selection cases cover root, intermediate, and cwd `.winterrc` files, per-role child overrides, and inheritance
  through files lacking the requested role; each selected version retains the path of the file that supplied it.
- Cases cover a Workspace configuration pin overriding the inherited Workspace selection, and Hub/Runner dispatch
  ignoring even an invalid `.winter/config.toml`.
- Cases cover an absent role, malformed or unreadable ancestor files even when shadowed, and a home-directory file both
  on and off the ancestor path.
- Cases cover independent user-selected defaults outside a workspace, directory selections overriding defaults, and a
  missing selected release refusing fallback to another installed release.
- Legacy fixtures cover every allowed command, enabled/disabled/missing/malformed flags, nearest-workspace discovery,
  leading and unknown Python options, raw native arguments, and explicit modern groups bypassing legacy configuration.
- Routing cases cover Python options before the command with both separate and equals-form values, stop scanning after
  the routing token, and prove root help/version work without any configuration or usable cwd.
- Source-override fixtures use separate target and implementation trees and prove the target's flag controls permission,
  only the initial `--winter=PATH` control is consumed, and all remaining native arguments keep their order and bytes.
- Cases cover relative, absolute, spaced, and non-UTF-8 override paths, malformed/empty controls, missing source
  directories, disabled targets with enabled source configs, and modern groups rejecting a launcher-level override
  without resolving its path or preparing Python.
- Override fixtures assert selected-source dependency inputs, venv/stamp and PATH, target-anchored process cwd, and
  original invocation cwd; invalid sources never cause fallback or bootstrap config creation.
- A stub `uv` and fixture venv entrypoint cover cold and warm starts, each dependency input becoming newer than the sync
  stamp, relocated venvs, synchronization failure, and cancellation during synchronization without orphaned processes.
- Fixture entrypoints assert the legacy cwd/environment adjustments and original invocation directory, stream purity,
  exit status, and signals; a decoy `winter` on PATH must never run.
- Bootstrap fixtures cover absent and existing config, root-only eligibility, preservation of operator content, and
  partial failure with and without a Python source override, without invoking real Workspace reconciliation.
- Filesystem fixtures isolate the complete root-to-cwd discovery tree, including ancestors above the fixture directory;
  verification never reads or modifies real ancestor selections, the user's version cache, or workspace pins.

## Isolate legacy launch preparation (engineering:legacy-adapter-boundary)

The legacy adapter owns Python source selection, environment preparation, and temporary bootstrap separately from
modern component resolution, with filesystem and synchronizer effects behind injected boundaries. Serves
`product:legacy-workspace-dispatch` and `product:legacy-init` under `decision:singleton-launcher-succession`.

Acceptance:

- Modern dispatch and platform help/version neither locate Python nor invoke `uv`, mutate legacy configuration, or
  prepare a virtualenv.
- Legacy synchronization uses native argument vectors without a shell; subprocess setup and cancellation preserve
  signal behavior and terminate and reap any synchronizer children before the launcher exits.
- All launcher effects outside process replacement are limited to the legacy adapter's documented dependency setup and
  init path; no Rust release cache, role default, or workspace pin is written.
