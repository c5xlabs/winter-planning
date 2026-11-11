---
name: hosted-runners
depends-on:
  - host-settings
  - sealed-tokens
code:
  - blizzard/src/blizzard/runner/cli/runtime.py::host
  - blizzard/src/blizzard/runner/cli/runtime.py::_RunnerThreads
  - blizzard/src/blizzard/runner/cli/store_guard.py::store_write_guard
  - blizzard/src/blizzard/runner/cli/store_guard.py::configured_ids
  - blizzard/src/blizzard/runner/lifecycle/identity_check.py::foreign_store_reason
  - blizzard/src/blizzard/runner/composition.py::add_runner
  - blizzard/src/blizzard/runner/composition.py::RunnerProcess.close_runner
  - blizzard/src/blizzard/runner/hub/identity_service.py::RetiredRunnerWatch
  - blizzard/src/blizzard/runner/cli/runners.py::add
  - blizzard/src/blizzard/runner/cli/runners.py::retire
  - blizzard/src/blizzard/runner/cli/runners.py::ingest
---

# Runners on a running host

Lets the API add and retire a host's runners: they become records the host tick reconciles, and the store's own
`hosted_runners` decides which store is the host's ([host-records.md](../spec/host-records.md) §Runners, §Adding a
runner, §Starting a runner, §Retiring a runner; [runner-store.md](../spec/runner-store.md) §Which store is the host's,
§The reader, §Seeding step 4). Delivers `product:runners-come-and-go` and `product:runner-id-fixed`; bound by
`engineering:deprecations-outlive-epic`, and shaped by `decision:config/runtime/walk-back-tokens`.

## Change

- `hosted_runners` and `hosted_runner_facts`, host-scoped, and their seed.
- `host_store_verdict` judges every boot and every `store_write_guard`, replacing each caller's `configured` ids.
- `composition.start_runner`: the per-runner sequence the boot runs, called by the boot and by the host tick for a
  runner added since the last tick.
- The add and retire routes; the host tick's reconcile of the set of runners; one runner's drain alone.
  `RetiredRunnerWatch` writes the `retired` fact.
- The app's federation, role policy, and public origins read from the host's `boot` keys, not its first runner.
- `runner add` and `retire` reach a running host, and write the store on a stopped one; neither edits `.env` or the
  file. `runners/<name>/` deprecated.
- Ingest writes the destination's hosted row and `added` fact.
- The operator pages on adding runners and on the host's runtime directory change to match. The upgrade note says a
  walk-back strands each runner added through the API until it is retired and added again
  (`engineering:contracts-follow-code`).
- Context: `bzh:crash-exemptions-runner` records the add's window. `runner-scoping.md` and `crash-correctness/runner.md`
  say a runner's binding is made from `hosted_runners` (`engineering:context-rules`).

## Acceptance

- `blizzard:unit-test` over `host_store_verdict`:
  - a store from before the epic judged against the file, as today;
  - a store hosting a runner the file never names judged ours;
  - a store sharing none of the file's runners refused;
  - an identity row with no hosted row refused.
- `blizzard:component-test` over:
  - an add starting the runner at the next tick and leaving its neighbours' ticks unbroken;
  - a host with a runner added through the API restarting and running every runner;
  - a runner released at the hub before its first tick claiming nothing;
  - a `start_runner` step failing for an added runner, its neighbours ticking and the start retried;
  - a forced retire draining one runner's sessions and no other's;
  - the host's first runner retired with the page still served;
  - a hub-side retirement stopping the runner's threads.
- `blizzard:service-test` adding a runner to a stepped host, restarting it, and claiming with the runner.
- `blizzard:crash-sweep` with crash points inside the add, the invariant checker passing at each.
- `blizzard:store-contract`, `blizzard:cli-contract`, `web:client-drift`, and `blizzard:compat` pass.
- `blizzard:manual-runner`.
