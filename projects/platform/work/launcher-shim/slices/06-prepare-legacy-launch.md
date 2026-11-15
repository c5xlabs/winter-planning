---
name: prepare-legacy-launch
depends-on:
  - select-legacy-source
  - lease-legacy-venvs
---

# Prepare legacy dependencies, bootstrap config, and the final exec request

Complete `LegacySetup` and `run_legacy` using legacy [Interfaces](../spec.md#interfaces), [Flows](../spec.md#flows), and
[Data](../spec.md#data). Deliver the setup/bootstrap portions of `product:legacy-workspace-dispatch` and
`product:legacy-init`, plus `engineering:legacy-adapter-boundary` and `engineering:fixture-only-dispatch`, under
`decision:singleton-launcher-succession` and `decision:launcher-shim/venv-ownership`.

## Change

- Compose validated target/source selection with shared warmth checks, exclusive stale setup, and a final shared-lease
  freshness recheck. A changed post-setup state is `Busy`, not an unbounded resynchronization loop.
- Resolve uv on inherited PATH only when synchronization is still needed under exclusive ownership. Invalidate the old
  stamp before invoking `uv sync --project <absolute selected-cli>` without a shell or locked/frozen flags.
- Run uv at target-cli with the specified setup environment and absolute `UV_PROJECT_ENVIRONMENT`; send both setup
  streams to stderr, propagate failure/interruption, and terminate/reap setup children before releasing ownership.
- Publish a source-aware stamp only after successful sync. While holding the shared lease, bootstrap creates only an
  absent target config and rewrites the recorded init token to `ws init`.
- Build the absolute venv entrypoint request with target-cli cwd, native inherited environment adjustments,
  launcher-entry timing, original invocation cwd, and the shared lease moved into `inherited_fds`.
- Map setup and replacement failures to the specified legacy diagnostics/statuses without selecting another source,
  invoking a wrapper, or exposing this path at the public CLI yet.

## Acceptance

- Injected filesystem/synchronizer tests prove source validation precedes writes, warm paths never locate uv, stale
  paths reread under exclusive ownership, and a newly warm recheck skips sync.
- Stub uv receives the exact native argv, target cwd, resolved venv, and permitted inherited uv configuration. Its
  fixture lockfile/cache writes remain allowed while Rust cache, pins, defaults, and installed-command decoys stay
  untouched.
- Failed/cancelled sync leaves the stamp absent, prevents Python launch, preserves the nonzero status or signal, and
  terminates/reaps all owned setup children. Invalidation/publication failures also prevent execution.
- Process doubles observe removal of `PYTHONHOME`/`WINTER_INVOKED_AS`, replacement of invocation cwd/launch time,
  `VIRTUAL_ENV`, venv-bin PATH prepending, target-anchored cwd, unchanged legacy args, and the held shared descriptor.
- Bootstrap fixtures preserve existing config bytes, create an absent empty config without enabling legacy, and change
  only the init token. Setup/Python failures never delete existing operator content.
- `Busy` is bounded and actionable; missing final entrypoints map to legacy exit 127, other replacement errors to 126,
  and no failure retries PATH `winter`, the wrapper, or a different implementation.

## Out of scope

- `classify-native-routes` owns recognition and removal of the optional first source-control token.
- `wire-launcher-dispatch` supplies real launcher inputs; `prove-dispatch-boundaries` covers cross-process concurrency
  and cancellation evidence.
