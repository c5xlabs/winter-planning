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
- Modern dispatch receives its native inherited environment through the injected request context; fixtures assert
  unrelated variables survive byte-for-byte and only `WINTER_INVOKED_AS` is replaced, without reading global environment
  from dispatch or the process adapter.
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
  legacy selection, and prove bare invocation, verbosity-only help, `-h`/`--help`, and `-V`/`--version` work without
  configuration or usable cwd.
- Routing fixtures put `init`, modern role names, platform controls, and `sample` in separate Python option values; they
  must reach flag-gated Python unchanged, never trigger bootstrap writes, Rust dispatch, or the sample adapter.
- Source-override fixtures use separate target and implementation trees and prove the target's flag controls permission,
  only the initial `--winter=PATH` control is consumed, and all remaining native arguments keep their order and bytes.
- Cases cover relative, absolute, spaced, and non-UTF-8 override paths, malformed/empty controls, missing source
  directories, disabled targets with enabled source configs, and modern groups rejecting a launcher-level override
  without resolving its path or preparing Python.
- Override fixtures assert selected-source dependency inputs, venv/stamp and PATH, target-anchored process cwd, and
  original invocation cwd; empty CLI directories, missing/unreadable entry modules, and missing/malformed/wrong-project
  metadata fail before synchronization or bootstrap config creation, even with a warm venv.
- A stub `uv` and fixture venv entrypoint cover cold and warm starts, each dependency input becoming newer than the sync
  stamp, relocated venvs, synchronization failure, and cancellation during synchronization without orphaned processes.
- Shared-venv fixtures switch A to B and back with older dependency mtimes, assert the source identity in the stamp,
  and prove each switch synchronizes before execution. Empty pre-Rust stamps also require synchronization.
- Overlapping fixtures cover concurrent warm readers, competing cold launches, and A/B source switches. Synchronization
  requires exclusive ownership; the post-setup shared-lease check prevents execution after an intervening source switch.
- A fixture entrypoint retains its shared lease across exec while another invocation attempts synchronization; normal
  exit, signals, and replacement failure release the lease. Synchronizer children do not retain the lease descriptor.
- Venv aliases resolve to one lease identity; the lockfile remains outside the replaceable venv, is never unlinked, and
  lock contention produces a bounded retry diagnostic without changing the environment or stamp.
- Failed/cancelled synchronization invalidates prior success evidence; a retry cannot reuse a partially changed venv as
  warm. Stamp-publication failure prevents execution.
- Stub synchronization may update a fixture lockfile and isolated uv cache; assertions distinguish these allowed legacy
  effects from forbidden writes to Rust selections, released components, or the installed command.
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
  init path, including its stable venv lockfile and uv's lockfile, cache, build, and managed-Python effects; no Rust
  release cache, role default, or workspace pin is written.
