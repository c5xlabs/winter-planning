# Runtime engineering

Derives from `epic:config/runtime`.

## What already holds

- The runner host reads `blizzard-host.toml` once, freezes a `LoopConfig` per runner on its first tick, ticks every
  runner and itself on an interval fixed at boot, and serves its runners from one process whose local API already
  carries human-lane writes (pause, takeover, the workspace prompt) over a unix socket and a federated session.
- The hub gave its tenants' records revisions, a change log, a declarative apply, and a read-on-use cache under
  `epic:live-config` (`bzh:configured-record`, `bzh:config-apply`, `bzh:config-read-on-use`). Its own operational
  settings are still bound into long-lived objects when it starts.
- Every signed-in human may call every human-lane write on the runner today; no runner route checks a role.

## Brief requirements

- engineering:no-renamed-settings — No setting's name changes. Serves `product:declared-lifetime`.
- engineering:running-session-undisturbed — No running session is ever disturbed by a settings change. Serves
  `product:session-lifetime`.
- engineering:one-hub-process — The hub's settings wake their sweeps in-process, so they assume the hub runs as one
  process; running it as several is outside this epic.
- engineering:unedited-daemon-unchanged — A host or hub never edited through its API behaves as it did before the epic,
  once its file has seeded it. Serves `product:runner-file-seeds` and `product:hub-file-seeds`.

## Off the fleet wire (engineering:off-fleet-wire)

Neither half crosses the fleet wire. Each daemon's settings are its own, a runner declares to the hub only what it
already declares at each registration, and neither daemon reads or checks the other's settings, so `blizzard-mock` does
not change and `bzh:wire-change-extends-mock` is not engaged. A host carrying any change of this epic works against any
hub in its supported version window, and a hub carrying them works against any runner in it. Serves
`product:independent-daemon-settings`.

## Contracts follow the code (engineering:contracts-follow-code)

The new routes regenerate the runner's and the hub's clients (`bzh:generated-client`), and the store, CLI, and
configuration contract documents are regenerated with the code that changes them. `docs/versioning.md` §What counts as
breaking gains the lifetime row, and `blizzard:compat` reports a key whose lifetime moved later as a break. Every change
to an operator surface regenerates its contract documents, carries its change note (`blizzard:change-notes`), and
updates the operator pages it touches: the pages on the runner host's configuration, adding runners, its runtime
directory, and its credentials, and the pages on the hub's configuration and config documents gain lifetimes, entries,
the seed, and the API.

## Context rules follow the code (engineering:context-rules)

The epic changes `blizzard-context` as well as `blizzard`:

- `bzh:config-key-declared` gains the lifetime and the `env` rule, and a sentence per daemon making a `pass` or
  `session` key read through `ConfigView` outside the seed a finding.
- `bzh:config-read-on-use` binds the runner's and the hub's entries, and its scope sentence on the hub's process
  settings narrows to the `boot` keys.
- `bzh:configured-record` extends to the host's records and gains the hub-wide record.
- `bzh:config-apply` extends to the runner's document.
- A new `bzh:settings-snapshot-per-pass` states the evaluation every pass makes, binding the runner and then the hub.
  Its detect list names an entry read after a pass has begun, an entry captured into an object at composition, and a
  `pass` or `session` key read from the environment.
- The domain's statement in `execution/hosts.md` §Bindings stops halting a runner for a retired workspace or
  subscription.
- `bzh:crash-exemptions-runner` records the window of a runner's add, and `runner-scoping.md` and
  `crash-correctness/runner.md` say a runner's binding is made from `hosted_runners`.

A change to `blizzard-context` lands with the `blizzard` change it describes, proven by
`blizzard-context:reference-lint`, `blizzard-context:markdown-format`, `blizzard-context:markdown-lint`, and
`blizzard-context:registry-drift`, and no change cites a rule that has not landed.

## Paired settings checked at their own daemon's write (engineering:paired-settings-checked)

A pair of settings inside one daemon is checked at that daemon's write once both sides are settings: the runner's
`api/hub_proxy.py` retry ceiling below the worker's read timeout, the heartbeat retention window above
`HEARTBEAT_STALENESS_THRESHOLD`, and the close-drain interval equal to its backoff base. Pairs across the two daemons
are not checked. Serves `product:independent-daemon-settings`.

## Migration on upgrade (engineering:upgrade-migration)

Each change's store revisions are applied with `blizzard runner migrate` or `blizzard hub migrate`
(`bzh:manual-migrations`). The first start after an upgrade seeds the store from the file; from then on the file is read
only for what the store lacks, and each upgrade note says so. An operator who shortens the tick with
`BZ_RUNNER_TICK_SECONDS` moves it to `tick_seconds` in the file before the variable is removed. Serves
`product:runner-file-seeds` and `product:hub-file-seeds`.

Acceptance:

- An upgraded host seeded from its file runs exactly as before.
- An operator who edits a seeded key in the file and restarts sees nothing change.

## Deprecations outlive the epic (engineering:deprecations-outlive-epic)

The surfaces this epic deprecates stay readable when it is delivered: `BZ_RUNNER_TICK_SECONDS`; the hub's
`work_source[]` key; `.env`'s runner tokens, `[hub_tokens]`, and `runners/<name>/`; and the `workspace_prompt` table
with its seed path. Removing each is a standalone breaking change belonging to no epic, made when an operator decides a
release may carry it. The default workspace is not among them: this epic removes it outright
(`decision:config/runtime/no-default-workspace`).

## Open questions

- Whether a runner needs its own thread group. Engineering.
- Whether a runner's id should keep scoping the store the way a tenant's does, rather than being an ordinary entity's
  key. Engineering.
