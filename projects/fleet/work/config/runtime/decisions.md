# Runtime decisions

## Tokens after a walk-back (decision:config/runtime/walk-back-tokens)

**Decider:** owner

**Open.** Once runner tokens are sealed in the host's store, a runner added through the API keeps its hub token only
there, so a host walked back to a release that reads tokens only from `.env` or `[hub_tokens]` strands every runner
added since the upgrade, recoverable only by retiring it from the hub and adding it again. The owner accepts this for
now and does not like it; before the sealed-token change is filed, decide whether the store's tokens are also written
somewhere the old release reads, or whether the walk-back is refused while such a runner exists. It blocks
`product:runners-come-and-go`.

## Walking back a release (decision:config/runtime/walk-back)

Walking a host or a hub back to a release that predates one of this epic's changes returns it to reading its file for
every key that change made an entry, leaving every change made through the API since in the store, unread. That is
accepted; an operator who means to walk back exports first and copies what they want into the file.

## No default workspace (decision:config/runtime/no-default-workspace)

**Decider:** owner

The `workspace-local` fallback and `init`'s scaffolded workspace are removed outright, as a break without a deprecation
window, because no deployed host relies on the fallback: every host `init` scaffolded carries its `[[workspace]]` in its
file, and it is seeded as any other.
