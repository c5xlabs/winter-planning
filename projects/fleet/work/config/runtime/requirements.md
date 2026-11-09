# Runtime requirements

Requirements of `epic:config/runtime`.

## Every setting has a lifetime (product:declared-lifetime)

Every setting has a lifetime: how long a value, once loaded, is lived with before the setting is evaluated again. The
lifetime is what an operator reads to know when a change they make will bite, and each setting declares it in the same
registry that declares its type and its default, so the answer to "can I change this live, and when will it bite?" is a
lookup.

Acceptance:

- Every setting of both daemons declares exactly one of four lifetimes: `pass`, `session`, `boot`, or `immutable`.
- The contract documents that already pin every key carry its lifetime.
- A limit a protocol, a peer, or another setting depends on is not a setting at all and has no lifetime: it is pinned in
  code rather than offered.

## Evaluated each pass (product:pass-lifetime)

Both daemons loop, and each time round they make a pass over what is in front of them. A `pass` setting is evaluated at
the start of every pass and lived with until the pass ends: load the settings, make the pass, load the settings again.

Acceptance:

- A change made while a pass runs waits for the next pass.
- Nothing is read halfway through a pass, so no pass weighs a budget it read at the start against a pace it read at the
  end.
- A request to the hub is a pass of its own, evaluating its settings when it arrives.

## Evaluated each session (product:session-lifetime)

A worker is the coding agent a runner launches into a session to work one step, and it is handed its model, its prompt,
its environment, and its harness settings when it starts; nothing reaches inside a running process to change them. A
`session` setting is evaluated when a worker is launched and lived with for that worker's whole session, because
changing a worker's world under its feet is a different and worse kind of surprise than changing it between sessions.

Acceptance:

- A change reaches every session launched after it.
- A session already running keeps what it launched with.

## Evaluated at boot (product:boot-lifetime)

A `boot` setting is evaluated when the daemon starts and lived with until it stops. These are a daemon's footing rather
than its behavior, and changing one is a restart of the daemon, honestly said.

Acceptance:

- A `boot` setting is the only kind an environment variable may supply, and where one does it overrides the file.
- A `pass` or `session` setting is never read from the environment, since a value fixed at process start would quietly
  make a live setting a `boot` one.

## Fixed at creation (product:immutable-lifetime)

An `immutable` setting is evaluated once, when the thing it belongs to is created, and never again, not even across a
restart: changing it would make a different thing.

Acceptance:

- A change to an `immutable` setting is refused; the thing is retired and a new one created instead.

## A runner's settings in its own store (product:runner-settings-in-store)

Every runner setting whose lifetime is `pass` or `session` becomes an entry in the host's store, revised rather than
overwritten, with a log of each change and who made it. The store, not the file, is the answer to what a runner is
configured with.

Acceptance:

- Each change to a runner's settings appears in its change log with who made it.

## Changed through the runner's API (product:runner-settings-api)

The runner's local API reads and writes its settings, and its CLI and its local page speak that API. A runner needs
nothing from the hub to change its own settings, so it can be tuned while the hub is out of reach.

Acceptance:

- Anyone the local page admits may read the settings; only the host's superuser may write them, since a setting such as
  a worker's environment reaches every session the runner launches.
- A write is checked in full against the same rules a boot applies to the file before any of it applies: a write that
  would be refused at boot is refused.
- After a refused write the runner keeps running exactly as it was and says what it rejected and why.
- A runner whose hub is out of reach still takes a settings change.

## The file seeds what the runner's store lacks (product:runner-file-seeds)

The file is how a new runner, or a setting new to an existing one, gets its first value; after that the API owns it. An
operator who keeps their host file in version control still stands a fresh runner up from it.

Acceptance:

- At startup, every setting the file states that has no entry in the store yet is created from the file.
- A setting the store already holds is left alone, and the file's value for it is ignored.

## Throttling, spending, and capacity live for one tick (product:tick-lived-settings)

The throttle's budget, pace, schedule, and utilization floor, what the runner does on a decline or a usage limit, the
cost caps per chunk and per runner, how many agents it runs, which gates it holds, which projects it serves, and whether
its queue is strict are all `pass` settings, and the tick is the pass that reads them. These are the settings the
engineer reaches for in the middle of a night, and a tick is short enough that the next one is soon.

Acceptance:

- A change to any of them is loaded at the start of the next tick and held for the whole of it.
- The runner tells the hub what it now accepts at that tick's registration, so the queue the hub hands out reflects it.
- Lowering a runner's agents never kills a session to make the number true: the runner stops claiming until it is under
  the line.

## Every pass, not only the tick (product:every-pass-evaluates)

Each runner runs its tick and, where tracing is on, a trace sweep beside it, and the host runs one tick of its own
across all its runners: the pass that reports to the hub, samples subscription usage, and starts and stops runners.

Acceptance:

- Each of these passes evaluates its own `pass` settings at its start, and so does each step inside it, the transcript
  drain and retention among them.

## What a worker is handed lives for its session (product:session-settings)

Model and effort aliases, the harness settings, a worker's environment, its shell and path, and the runner and workspace
prompts are `session` settings. A session records which settings it launched under, so the engineer comparing two runs
of the same chunk can see which side of an edit each fell on.

Acceptance:

- A change to any of them applies to every session started after it, and never to one already running.
- Each session shows which settings it launched under.

## The loop's own cadence as a setting (product:tick-interval-setting)

The tick's interval becomes a setting like any other, changed through the runner's API and seeded from the file. Its
lifetime is `pass`: each tick evaluates the interval to the next, so a runner can be sped up or slowed down while it
runs.

Acceptance:

- The environment variable that sets the interval today is deprecated, since an environment variable may supply only a
  `boot` setting; until it is removed, it seeds the setting only when neither the store nor the file holds one.

## Every setting of a runner, its name included (product:runner-settings-all-live)

Nothing inside a runner's own configuration waits for a restart. The name is a label the runner declares at each
registration, not its identity, so its lifetime is `pass` like the rest.

Acceptance:

- A runner renamed through its API declares the new name at its next registration, with no restart.

## A runner's id never changes (product:runner-id-fixed)

A runner's id is `immutable`. The hub mints it when the runner is added, every lease and record the runner holds is
keyed by it, and every worker reaches its runner by a path that carries it.

Acceptance:

- A runner wanted under another id is retired and a new one added.

## Runners that come and go on a running host (product:runners-come-and-go)

Today adding a runner waits for the host's next start, and a retired runner's threads run on after the hub refuses its
token. A runner's hub token moves from a file keyed by its name into the host's store, keyed by its id and encrypted, so
nothing about a runner but its id outlives a rename.

Acceptance:

- A runner added to a host starts at the host's next tick, with no restart of the host or of the runners beside it.
- A runner retired from a host stops at the host's next tick.
- A renamed runner still finds its hub token.

## The host's footing lives for the host's boot (product:host-footing)

The host's address and port, which every running worker was handed to reach its runner by; its store, where every other
setting now lives; the hub it reports to and the host token it presents there; and its auth are the ground every runner
on the host stands on. Their lifetime is `boot`: they rarely move, and moving them under running sessions is the one
change that cannot be made safe cheaply.

Acceptance:

- They are not entries: they stay in the file, read at boot as today, and changing one restarts the host.

## Workspaces and subscriptions as records (product:host-records)

The workspaces a host provisions into and the subscriptions its runners spend become records of the host, the way forges
and repositories are records of the hub: created, edited, and retired through the host's API, never deleted, and taken
up at the host's next tick. A host has no default workspace: it has the workspaces its operator declares or creates, and
none until then.

Acceptance:

- A workspace's id and a subscription's slug and provider are `immutable`, since the hub records the workspace a claim
  worked in and usage history and limit marks are kept by subscription; one wanted under another id is retired and a new
  one created.
- Retiring one stops new work through it: the host stops reporting it, and the hub withholds work from a binding naming
  a workspace the host does not report, and from a runner naming a subscription outside the host's roster.
- A runner bound to a retired workspace keeps working its other bindings.
- A retired record's history stays.

## A workspace is the operator's to reshape (product:workspace-reshape)

Every setting of a workspace other than its id (its provider, its root, its repositories, its environment pool and
limit, its base branch) can change at any time, with nothing refused, and applies to the next environment acquired.
Blizzard does not reconcile what is on disk: an operator who repoints a workspace adjusts the disk to match, outside
blizzard.

Acceptance:

- An environment already held is released and resumed through the workspace as it now stands.

## The settings in force, visible (product:settings-in-force)

"What is this runner actually running with" is answered by asking the runner, not by reading a file the runner may have
outgrown. The revisions stay on the runner: the hub is told what a runner accepts, not which revision it runs.

Acceptance:

- The runner's status and its local page show which revision of its settings each runner is running and when it took
  hold.
- Each session shows which revision it launched under.

## A runner's whole document, applied (product:runner-settings-document)

Beyond one setting at a time, the runner takes a document shaped like its file and reconciles its store to it in one
validated write. Its export is the same document, so a host's settings can be kept in version control and applied back.

Acceptance:

- Every setting the document states is set, and every workspace or subscription it names is created or edited.
- Nothing the document leaves out is retired.

## The hub's operational settings as a record (product:hub-settings-record)

`epic:live-config` gave the hub records that are revised rather than overwritten, a log of every change and who made it,
an API its CLI and board both speak, and a board to read and edit them on; the hub's own behavior still lives in
`blizzard-hub.toml`. The cadence and budgets of the hub's background passes, its transcript caps, the egress and
trace-export settings, the login throttle, and the produces and route-token modes become configuration held in the hub's
store and changed through the hub's API, under the same revisions, change log, and board that work sources already have.

Acceptance:

- They are the hub's, shared by every tenant, so they are one hub-wide record changed only by a hub administrator, never
  a record of any tenant.

## The hub's passes pick up their settings each time round (product:hub-pass-settings)

A background pass of the hub loads its interval, its budget, and every other setting it uses when it starts, not when
the hub started, and holds them until it ends. A request does the same when it arrives.

Acceptance:

- A change to a hub setting reaches the next pass or request without a restart of the hub.

## The file seeds what the hub's store lacks (product:hub-file-seeds)

The hub follows the runner's rule. Its file keeps, as its own, what is genuinely the process's: the address it listens
on, its store, its auth providers, and the key its secrets are sealed under.

Acceptance:

- At startup, a setting the hub's file states with no entry in the store is created from the file, and one the store
  already holds is left alone.

## The hub's whole document, applied (product:hub-settings-document)

As on the runner, the hub's settings take a document shaped like its file, applied in one write and exported back.

Acceptance:

- The hub's settings document is separate from any tenant's config document.

## Each daemon sets its own side (product:independent-daemon-settings)

A few settings only make sense beside a partner on the other daemon: the hub's offline cutoff against the runner's tick,
its usage staleness against the runner's sample interval, its transcript record cap against the runner's. Each daemon's
settings are its own, and both are free to set their own side.

Acceptance:

- Neither daemon checks its side against the other's; a runner and a hub set out of step behave as their settings say.
