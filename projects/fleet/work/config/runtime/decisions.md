# Runtime decisions

## An export carries values only (decision:config/runtime/export-values-only)

**Decider:** owner

Either daemon's export writes only the entries that hold a value; an entry `at_default`, a setting unset back to its
default, is left out, and no document unsets a setting. A document only ever sets, as it already never retires, and it
keeps its file's shape: a reserved marker for an unset would be grammar no file accepts, and TOML cannot state a null
default. Applying an export back to its own daemon still writes nothing; on another daemon such a setting keeps the
value it has, and an operator resets it there with `settings unset`.

## Tokens after a walk-back (decision:config/runtime/walk-back-tokens)

**Decider:** owner

A runner added through the API has its hub token only in the store, and a release from before this epic reads tokens
only from `.env` and runners only from the file. Walking a host back to such a release strands every runner added
through the API: the older release does not run it, and the hub still holds it. The strand is accepted. Neither daemon
writes tokens anywhere an older release reads, and no walk-back is refused. The operator recovers a stranded runner by
retiring it at the hub and adding it again. Writing a second copy of the token would bring back the `.env` writes this
epic removes, and refusing a walk-back would mean an older release has to enforce a rule it predates. The upgrade note
names the strand beside `decision:config/runtime/walk-back`.

## Settings pulled per pass (decision:config/runtime/pull-per-pass)

Neither daemon watches its store or pushes a write into a running object; the next pass finding a moved revision is the
only way a change arrives, because that keeps a cold start and a long-lived process equally correct.

## A runner's settings are the runner's (decision:config/runtime/runner-scoped-settings)

A runner's settings tables are runner-scoped rather than host tables, so `blizzard runner` transfer and ingest carry a
runner's settings with it to another host: what a runner is configured with is the runner's, not the machine's.

## Hub settings reads need the permission (decision:config/runtime/hub-settings-admin-reads)

Reading the hub's settings takes `SETTINGS_ADMIN` as writing does, because the hub's settings are no single tenant's
business.

## Walking back a release (decision:config/runtime/walk-back)

Walking a host or a hub back to a release that predates one of this epic's changes returns it to reading its file for
every key that change made an entry, leaving every change made through the API since in the store, unread. That is
accepted; an operator who means to walk back exports first and copies what they want into the file.

## No default workspace (decision:config/runtime/no-default-workspace)

**Decider:** owner

The `workspace-local` fallback and `init`'s scaffolded workspace are removed outright, as a break without a deprecation
window, because no deployed host relies on the fallback: every host `init` scaffolded carries its `[[workspace]]` in its
file, and it is seeded as any other.
