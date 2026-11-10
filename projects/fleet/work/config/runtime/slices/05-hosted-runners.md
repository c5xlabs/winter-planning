---
name: hosted-runners
depends-on:
  - shared-sealing
  - host-settings
code:
  - blizzard/src/blizzard/runner/cli/runtime.py::_RunnerThreads
  - blizzard/src/blizzard/runner/composition.py::add_runner
  - blizzard/src/blizzard/runner/composition.py::RunnerProcess.close_runner
  - blizzard/src/blizzard/runner/composition.py::held_hub_token
  - blizzard/src/blizzard/runner/hub/held_token.py::HubTokenHolder
  - blizzard/src/blizzard/runner/hub/identity_service.py::RetiredRunnerWatch
  - blizzard/src/blizzard/runner/host_config.py::resolve_token
  - blizzard/src/blizzard/runner/cli/runners.py::add
  - blizzard/src/blizzard/runner/cli/runners.py::retire
  - blizzard/src/blizzard/runner/cli/runners.py::ingest
  - blizzard/src/blizzard/runner/composition.py::DotenvTokenFiles
---

# Runners on a running host

Makes the host's runners records added and retired through its API and reconciled by the host tick, with their hub
tokens sealed in the store by id ([host-records.md](../spec/host-records.md) §Runners, §Adding a runner, §Retiring a
runner; [runner-store.md](../spec/runner-store.md) §Sealed tokens, §The reader, §Seeding steps 4 and 6). Delivers
`product:runners-come-and-go` and `product:runner-id-fixed`; bound by `engineering:deprecations-outlive-epic`. Waits on
`decision:config/runtime/walk-back-tokens`.

## Change

- `hosted_runners` and `hosted_runner_facts`, host-scoped, and their seed.
- `runner_hub_tokens`, the host's key and key directory, and the seed's import of every runner's token.
- The add and retire routes; the host tick's reconcile of the set of runners; one runner's drain alone;
  `RetiredRunnerWatch` writing the `retired` fact.
- The app's federation, role policy, and public origins read from the host's `boot` keys, not its first runner.
- `runner add` and `retire` reach a running host and write the store on a stopped one; neither edits `.env` or the file.
- `.env`'s runner tokens, `[hub_tokens]`, and `runners/<name>/` deprecated and named in the upgrade note.
- Ingest writes the destination's hosted row and `added` fact and re-seals the moving runner's token.
- The operator pages on adding runners, the host's runtime directory, and its credentials change to match
  (`engineering:contracts-follow-code`).
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
