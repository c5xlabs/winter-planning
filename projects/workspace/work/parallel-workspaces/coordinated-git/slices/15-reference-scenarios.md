---
name: reference-scenarios
depends-on:
  - ws-connect-disconnect
  - ws-pull
  - ws-merge
  - ws-push
  - ws-status-command
  - ws-diff
---

# Recorded scenarios of the coordinated Git commands

Records `ws connect`, `ws disconnect`, `ws fetch`, `ws pull`, `ws merge`, `ws push`, `ws status`, and `ws diff` against
the reference tool and replays them against the binary, proving the flows of this epic's spec end to end under
`decision:functional-parity`; each registered deviation carries its `[deviation]` decision and expected result.

## Change

- Scenarios under `scenarios/ws-connect/`, `ws-disconnect/`, `ws-fetch/`, `ws-pull/`, `ws-merge/`, `ws-push/`,
  `ws-status/`, and `ws-diff/`, recorded with `cargo dev record` from the reference tool, each with a fixture workspace
  whose environments and projects push to and fetch from fixture remotes; parallel events are compared per target, as
  `engineering:bounded-fan-out` asks.
- No fixture declares `nested`, so no status row carries the nested member.
- No production code changes.

## Acceptance

- Scenarios: connect then push a new branch; disconnect; fetch advancing a source main; pull fast-forwarding two
  environments; merge of `{main}`; push with nothing to push; status of a clean and a dirty workspace; diff in each
  mode.
- Deviation scenarios, each with its decision and `expected/`: a skipped pinned worktree in connect and in push under
  `decision:parallel-workspaces/coordinated-git/pinned-skips`; a conflicted autostash and a dirty blocker under
  `decision:parallel-workspaces/coordinated-git/integration-conflicts`; a failed refresh during pull under
  `decision:parallel-workspaces/coordinated-git/fetch-freshness`; an absent worktree named in `missing` and an unknown
  literal environment under `decision:parallel-workspaces/target-selection`.
- `cargo dev gate full` exits 0.
