---
name: hosted-runners
depends-on:
  - shared-sealing
  - host-settings
---

# Runners on a running host

Makes the host's runners records added and retired through its API and reconciled by the host tick, with their hub
tokens sealed in the store by id ([host-records.md](../spec/host-records.md) §Runners,
[runner-store.md](../spec/runner-store.md) §Sealed tokens). Delivers `product:runners-come-and-go` and
`product:runner-id-fixed`. Waits on `decision:config/runtime/walk-back-tokens`.

## Change

- The host-scoped hosted-runner tables and their seed.
- The sealed token table, the host's key and its key directory, and the seed's import of every runner's token.
- The add and retire routes; the host tick's reconcile; a runner's drain alone; `RetiredRunnerWatch` writing its fact.
- The app's settings read from the host rather than its first runner.
- `runner add` and `retire` reach a running host and write the store on a stopped one.
- `.env`'s runner tokens, `[hub_tokens]`, and `runners/<name>/` deprecated (`engineering:deprecations-outlive-epic`).
- Ingest writes the destination's hosted row and re-seals the moving runner's token.
- The operator pages on adding runners, on the host's runtime directory, and on its credentials change to match.
- Context: `bzh:crash-exemptions-runner` records the add's window; `runner-scoping.md` and `crash-correctness/runner.md`
  say a runner's binding is made from `hosted_runners` (`engineering:context-rules`).

## Acceptance

- `blizzard:unit-test`: a token sealed under one runner's id refuses to open under another's.
- `blizzard:component-test` over:
  - an add starting the runner at the next tick and leaving its neighbours' ticks unbroken;
  - a forced retire draining one runner's sessions and no other's;
  - the host's first runner retired with the page still served;
  - a hub-side retirement stopping the runner's threads;
  - a rename leaving the runner's token found;
  - an upgraded host whose tokens were in `.env` and `[hub_tokens]` finding every one in the store;
  - ingest moving a runner whose token re-seals.
- `blizzard:service-test` adding a runner to a stepped host and claiming with it.
- `blizzard:crash-sweep` with crash points inside the add and the token import, the invariant checker passing at each.
- `blizzard:store-contract`, `blizzard:config-contract`, `blizzard:cli-contract`, `web:client-drift`, and
  `blizzard:compat` pass.
- `blizzard:manual-runner`.
