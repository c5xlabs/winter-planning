---
name: prepare-cached-dispatch
depends-on:
  - resolve-role-versions
---

# Prepare and replace cached component executables

Implement modern `prepare`, `run_dispatch`, and failure replies from [Interfaces](../spec.md#interfaces) and the modern
[Flows](../spec.md#flows), using the manifest in [Data](../spec.md#data). Deliver `product:reach-component`,
`product:reuse-component-release`, `product:explain-dispatch-failure`, and the modern handoff portions of
`product:preserve-invocation` and `product:invocation-aware-help`, with `engineering:fixture-only-dispatch`, under
`decision:launcher-shim/component-discovery` and `decision:version-cache`.

## Change

- Locate only `<cache>/winter/versions/<role>/<version>/<target>/component.toml`; validate role, exact version, target,
  handoff version 1, and the contained relative executable path.
- Construct `ExecRequest` with unchanged native component arguments, inherited cwd, no extra descriptors, and the
  injected environment with only `WINTER_INVOKED_AS` replaced.
- Replace the process directly without executable existence/mode preflight, a PATH search, installation, or retries.
- Render selection/configuration/cache/manifest/handoff/replacement errors with the specified source paths, prefixes,
  empty stdout, and 126/127 exits. Leave this path unregistered at the public CLI.

## Acceptance

- Fixture manifests cover independent roles, versions, targets, shared reuse, and availability with all other roles
  absent. Cache contents never change the selected release.
- Missing manifests yield `NotCached`; malformed or mismatched manifests and escaping executable paths yield
  `InvalidManifest`; unsupported handoff versions yield `IncompatibleHandoff`. Unknown keys are ignored.
- Process doubles receive native arguments and environment unchanged except for invocation metadata; modern cwd is
  inherited and no lease descriptors are supplied.
- Controlled replacement failures, including missing/non-executable targets, remain modern exit 126 errors; absent
  selection/cache errors are 127. Diagnostics identify the role and available version/source context.
- Failure tests prove no lower-precedence release, PATH decoy, Python entrypoint, compiler, or network action runs, and
  configuration/cache snapshots remain unchanged.

## Out of scope

- `wire-launcher-dispatch` exposes the route; `prove-dispatch-boundaries` proves real process behavior and the receiving
  component's invocation-aware help contract with stand-ins.
