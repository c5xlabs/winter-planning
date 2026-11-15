---
name: select-legacy-source
depends-on:
  - production-process-contract
---

# Select and validate the target and Python implementation

Build the target/source portion of the legacy [Domain](../spec.md#domain) and [Flows](../spec.md#flows), serving
`product:legacy-workspace-dispatch`, `product:legacy-init`, `engineering:legacy-adapter-boundary`, and
`engineering:fixture-only-dispatch`. Follow `decision:singleton-launcher-succession` and
`decision:launcher-shim/component-boundary`; discovery remains separate from venv effects and public routing.

## Change

- Discover the ordinary legacy target from the nearest workspace config and require a boolean
  `enable-legacy-winter = true` plus the target's `tools/winter-cli/`.
- For bootstrap, require `cwd/tools/winter-cli/` without ancestor discovery or flag reads; defer configuration creation.
- Resolve a supplied source override from original cwd, canonicalize the selected CLI, and validate readable regular
  `pyproject.toml` and entry-module files with the specified project name and script entrypoint.
- Keep target/permission, original invocation directory, and selected implementation distinct. Do not read source-tree
  config for permission or consult Rust selections, XDG directories, or caches.
- Return the specified workspace, disabled, missing-CLI, invalid-config, and invalid-source failures before setup
  effects.

## Acceptance

- Fixture targets cover enabled, missing, false, non-boolean, malformed, and unreadable flags, plus nearest-workspace
  stopping without outer fallback. Disabled diagnostics name the flag and explicit modern entry point.
- Separate implementation trees with absolute, relative, spaced, and non-UTF-8 paths prove that overrides change only
  the source; an enabled source config cannot authorize a disabled target.
- Missing/unreadable/malformed/wrong-project metadata, wrong script entries, and absent/unreadable entry modules fail
  explicitly, even with a warm fixture venv. No fallback, uv invocation, stamp change, or bootstrap write occurs.
- Bootstrap fixtures reject ancestor-only eligibility and empty CLI directories before creating config, while a valid
  cwd target requires no flag or Rust version/default.
- Discovery runs against isolated filesystem adapters and does not import Python or execute live Workspace code.

## Out of scope

- Lease ownership and freshness belong to `lease-legacy-venvs`; synchronization, environment construction, and deferred
  bootstrap writes belong to `prepare-legacy-launch`.
