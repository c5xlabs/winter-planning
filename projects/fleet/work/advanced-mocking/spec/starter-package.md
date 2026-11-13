# Starter package

The fixtures every provisioned feature env starts from, applied as `fixture apply blizzard-mock/fixtures/starter/`. A
fresh agent opening the board, the runner panel, or the mock forge sees exactly this. It replaces `scenario board` and
`scenario fleet` (product:scenario-retirement, product:starter-world).

## System

| File                                                                         | Kind      | Holds                                                                |
| ---------------------------------------------------------------------------- | --------- | -------------------------------------------------------------------- |
| [`shell.yaml`](../assets/starter/shell.yaml)                                 | shell     | Two projects, three sources and repositories, urgencies, three people and an invitation |
| [`codebase-billing.yaml`](../assets/starter/codebase-billing.yaml)           | codebase  | `billing-service` and four commits of history                        |
| [`backlog.yaml`](../assets/starter/backlog.yaml)                             | backlog   | Labels, 25 issues, a hub-authored work item                          |
| [`graphs.yaml`](../assets/starter/graphs.yaml)                               | graphs    | `review-gated`, with a human gate, beside the packaged `bas-dwf`     |
| [`gardening.yaml`](../assets/starter/gardening.yaml)                         | gardening | Scopes, two routines, a weekly cadence; proposals in every closure   |
| [`runners.yaml`](../assets/starter/runners.yaml)                             | runners   | A host and seven runners in every presence and brake                 |
| [`activity.yaml`](../assets/starter/activity.yaml)                           | activity  | Nineteen chunks across all nine statuses; the event log              |

The env's own services stay as `winter service up` brings them: the hub's startup script still declares the mock forge,
`toy-api`, `toy-web`, and the `default` project. The shell fixture declares the same records again, so it is
self-contained, and the apply finds them unchanged.

### What it covers

| Surface         | Edge states present                                                                                                                                                                                                                     |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Chunk status    | All nine: `not_ready`, `ready`, `running`, `delivering`, `paused`, `waiting_on_human`, `needs_human`, `done`, `stopped`                                                                                                                 |
| Chunk shape     | Two work refs on one chunk; one chunk across two repositories; a blocked chunk behind a prerequisite; a hub-authored work item; a 180-character title; a non-ASCII title; three urgencies, one custom with an emoji                     |
| Waiting         | An open question with options; an answered question not yet delivered; a graph-declared human gate; a runner-imposed gate over red checks                                                                                               |
| Needs human     | Escalations for `retries-exhausted`, `spend-cap`, `forge-auth`, `bounce-cap`                                                                                                                                                             |
| Delivery        | An open pull request with checks pending; one with a failing check; a merged one; two conflict bounces on one chunk; one repository landed and the other not                                                                           |
| Cost            | Exact cost; a cost-partial chunk (a usage row with no cost); an estimated cost; a week of hourly usage                                                                                                                                 |
| Runners         | Online, offline 8 h, never connected, fleet-paused, locally paused with a reason, retired; two runners sharing a name; a host with two runners; a subscription at 96%; a usage sample miss; a declared empty roster; a declining lever |
| Runner panel    | The env's own runner holds a lease parked on an open ask, an open takeover, two env bindings, and a finalized and an open transcript                                                                                                    |
| People          | An admin, a contributor, a guest, an open invitation                                                                                                                                                                                    |
| Gardening       | Two routines on a weekly cadence; proposals open, passed, accepted and minted, accepted and declined                                                                                                                                   |
| Forge           | Issues never ingested; a closed-as-not-planned issue; a 14-comment thread; labels; a second project's repository with real history                                                                                                     |
| Event log       | All three severities, including `worker-lost`, `needs-human`, `forge-rate-limited`, `flow-landed`, and 30 generated rows                                                                                                                |

## Open questions

- Whether the starter package is applied by `winter provision` or by the first `winter service up`, since the hub, forge,
  and runner must be up before an apply.
- Whether `bas-dwf`'s node names are stable enough for `history` to name them, or the starter's activity uses only
  `review-gated`, which it owns.
