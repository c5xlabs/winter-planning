# Runtime decisions

## Tokens after a walk-back (decision:config/runtime/walk-back-tokens)

**Decider:** owner

**Open.** Whether the store's sealed tokens are also written somewhere an older release reads, or a walk-back is refused
while a runner added through the API exists, since such a runner's hub token lives only in the store and a walk-back
strands it until it is retired and added again. The owner accepts the strand for now; it is decided before the
sealed-token change is filed, and it blocks `product:runners-come-and-go`.

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
