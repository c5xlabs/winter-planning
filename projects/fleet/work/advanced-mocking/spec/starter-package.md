# Starter package

The fixtures every provisioned feature env starts from. After it brings the env's hub, mock forge, and runner up and
before it hands the env over, `winter provision` resets the env (`blizzard-mock-data reset --env <env>`) and applies
them (`fixture apply blizzard-mock/fixtures/starter/ --env <env>`). The reset comes first on every provision, so an env
provisioned before this change, whose hub holds the startup script's old records, and an env an agent has filled both
come back to exactly the starter world; provisioning discards whatever the env held. An apply that exits 2 or 3 fails
the provision, naming the fixture and the record, so a fresh agent opening the board, the runner panel, or the mock
forge sees exactly this. It replaces `scenario board` and `scenario fleet` (product:scenario-retirement,
product:starter-world).

## System

| File                                                                         | Kind      | Holds                                                                |
| ---------------------------------------------------------------------------- | --------- | -------------------------------------------------------------------- |
| [`shell.yaml`](../assets/starter/shell.yaml)                                 | shell     | The mock forge and its secret, two projects, three sources and repositories, urgencies, three people and an invitation |
| [`codebase-billing.yaml`](../assets/starter/codebase-billing.yaml)           | codebase  | `billing-service` and four commits of history                        |
| [`backlog.yaml`](../assets/starter/backlog.yaml)                             | backlog   | Labels, 27 issues, a hub-authored work item                          |
| [`graphs.yaml`](../assets/starter/graphs.yaml)                               | graphs    | `review-gated`, with a human gate, and `overflow`, beside the packaged `bas-dwf` and `garden-routine`     |
| [`gardening.yaml`](../assets/starter/gardening.yaml)                         | gardening | Scopes, two routines, a weekly cadence; proposals in every closure   |
| [`runners.yaml`](../assets/starter/runners.yaml)                             | runners   | A host and eight runners in every presence and brake                 |
| [`activity.yaml`](../assets/starter/activity.yaml)                           | activity  | Twenty-one chunks across all nine statuses; the event log              |

`shell.yaml` owns the env's shell records: the mock forge, its secret, every source and repository, and the projects.
The hub's startup script (`hub-host.sh` in the workspace's process-compose config) stops declaring them and only
initialises and starts the hub, so a reset env's hub holds no forge until a shell fixture is applied. The shell
fixture adopts the `default` project and `standard` urgency every new tenant holds, renaming them Storefront and
Normal.

The starter leaves the env's own runner locally paused, so it claims none of the ready chunks; an agent that wants its
runner to pick up work resumes it. Delivering chunks are held at their land node by pending checks for three hours after
the apply, then bounce as real ones would. The forge address
and the toy repositories' base branch come from the env's variables (`${BZ_FORGE_PORT}`, `${BZ_BASE_BRANCH}`).

### What it covers

| Surface         | Edge states present                                                                                                                                                                                                                     |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Chunk status    | All nine: `not_ready`, `ready`, `running`, `delivering`, `paused`, `waiting_on_human`, `needs_human`, `done`, `stopped`                                                                                                                 |
| Chunk shape     | Two work refs on one chunk; one chunk across two repositories; a blocked chunk behind a prerequisite; a hub-authored work item; a 200-character title; a non-ASCII title; three urgencies, one custom with an emoji                     |
| Waiting         | An open question with options; three open questions on one chunk; an answered question not yet delivered; a graph-declared human gate; a runner-imposed gate                                                                                               |
| Needs human     | Escalations for `retries-exhausted`, `spend-cap`, `forge-auth`, `bounce-cap`                                                                                                                                                             |
| Delivery        | Open pull requests with checks pending; a merged one; two conflict bounces on one chunk; failed-check bounces; one chunk with one repository landed and the other waiting on checks                                              |
| Cost            | Exact cost; a cost-partial chunk (a usage row with no cost); six hours of generated usage behind a spend cap                                                                                                                                 |
| Runners         | Online, offline 8 h, never connected, fleet-paused, locally paused with a reason, retired; two runners sharing a name; a host with two runners; a subscription at 96%; a usage sample miss; a declared empty roster; a declining lever; a name longer than any column |
| Runner panel    | The env's own runner holds a lease parked on an open ask, an escalated lease under an open takeover, and a finalized and an open transcript                                                                                                    |
| People          | An admin, a contributor, a guest, an open invitation                                                                                                                                                                                    |
| Gardening       | Two routines on a weekly cadence; proposals open, passed, accepted and minted, accepted and declined                                                                                                                                   |
| Forge           | Issues never ingested; a closed-as-not-planned issue; a 14-comment thread; labels; a second project's repository with real history                                                                                                     |
| Event log       | All three severities, including `worker-lost`, `needs-human`, `forge-rate-limited`, `flow-landed`, and 30 generated rows                                                                                                                |
| Overflow        | A runner name, a custom node name, and an artifact name each long enough to overflow any column (the `--stress` extremes) |
