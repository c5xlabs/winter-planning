# Runners, workspaces, and subscriptions on a running host

Tables are [runner-store.md](./runner-store.md) §Data; the host tick's reconcile is [passes.md](./passes.md) §The host
tick; every route here takes `require_superuser` ([runner-api.md](./runner-api.md) §Who may write).

## Domain

```mermaid
stateDiagram-v2
    direction LR
    state "Hosted runner" as R {
        [*] --> added: add, or seed of a [[runner]] with an id
        added --> retired: retire at the hub, or RetiredRunnerWatch
        retired --> [*]: final; the file cannot bring it back
    }
    state "Workspace or subscription" as W {
        [*] --> active: create, or seed
        active --> retired_w: retire
        retired_w --> active: enable
        note right of retired_w: never deleted; history stays
    }
```

- A hosted runner is the host's while its newest `hosted_runner_facts` row is `added`. Its id is hub-minted and
  `immutable`; everything else about it is its entries (`product:runner-id-fixed`).
- A workspace or subscription is a record in the shape `bzh:configured-record` gives the hub's: an `immutable` key
  (`workspace_id`; a subscription's `slug` and `provider`), a `revision` every write increments, a
  `host_setting_changes` row beside every write, sparse-merge edits, retirement as an appended fact, no delete
  (`product:host-records`).
- No write is gated on use: a workspace every runner binds can be retired, repointed, or moved to another provider, and
  a subscription every runner spends can be retired.

What each change does, taken up at the next host tick (`product:workspace-reshape`):

| Change                | Effect                                                                                                                                                                                                                                                                                                                   |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Create a workspace    | The host's from the next host tick; serves work once a runner binds a project to it                                                                                                                                                                                                                                      |
| Create a subscription | Sampled and reported from the next host tick                                                                                                                                                                                                                                                                             |
| Edit a workspace      | Provider, root, repositories, pool, limit, base branch, or prompt: a new provider; the next environment is acquired through it. A held environment stays held and is released or resumed through the workspace as it now stands. Blizzard does not reconcile the disk                                                    |
| Edit a subscription   | Name, credentials path, usage API base, or sample interval: sampler and renewer rebuilt. A running session keeps the credentials it launched with                                                                                                                                                                        |
| Retire a workspace    | Not reported. Each runner binding a project to it keeps its other bindings and claims nothing through it; the hub flags the binding unreported, as today; a chunk already claimed through it finishes there                                                                                                              |
| Retire a subscription | Not sampled or reported. A runner naming it declares none at registration from its next tick, is throttled by no subscription's usage, and is offered work as a runner paid for some other way. Its usage samples and usage-limit marks stay. Naming it again once enabled, or naming another, is the runner's own write |
| Enable either         | The retirement reversed                                                                                                                                                                                                                                                                                                  |

## Interfaces

### Runners

```http
POST /api/host/runners
{ "name": "night-1", "projects": [...], "settings": { "agents": 2 } }

→ the runner's view with "state": "starting"
422 settings_invalid       # `settings` validated as a write is, before the hub is asked: no mint at the hub
```

```http
POST /api/host/runners/{runner_id}/retire
{ "force": false }

→ the runner's view
# The hub's refusal while the runner holds a chunk, unless forced (execution/retirement.md), is the host's answer.
```

`GET /api/host/runners` lists the hosted runners.

| CLI                      | Host running     | Host stopped                                                                                             |
| ------------------------ | ---------------- | -------------------------------------------------------------------------------------------------------- |
| `blizzard runner add`    | The add route    | The same transaction against the store, holding the host's lock (`store_write_guard`); says which it did |
| `blizzard runner retire` | The retire route | As above; the host's next start takes the change up at its first tick                                    |

Neither verb edits `blizzard-host.toml` or `.env`. An operator who wants the file to hold a runner exports
([runner-api.md](./runner-api.md) §Apply).

### Workspaces and subscriptions

| Verb   | Route                                                                           | Notes                                          |
| ------ | ------------------------------------------------------------------------------- | ---------------------------------------------- |
| create | `POST /api/host/workspaces`, `POST /api/host/subscriptions`                     | Validated as the file is at boot               |
| list   | `GET` the same                                                                  | `?retired=true` includes retired records       |
| show   | `GET /api/host/workspaces/{workspace_id}`, `GET /api/host/subscriptions/{slug}` |                                                |
| edit   | `PATCH` the same, `If-Match`                                                    | Legal on a retired record, which stays retired |
| retire | `POST …/{key}/retire`                                                           | Of a retired record: writes nothing            |
| enable | `POST …/{key}/enable`                                                           | Of an enabled record: writes nothing           |

A create or edit is refused `422 settings_invalid` with each key's error for what the file would fail at boot: a
provider the host cannot build, a root that is not a path, a `max_environments` below one.

### The workspace prompt

`prompt`, `prompt_file`, and `prompt_package` are settings of the workspace record. The existing routes
(`runner/api/workspace_prompt.py`) stay and write the record through the same write as any edit:

| Route                                               | Writes                                                                                                |
| --------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `PUT /api/host/workspaces/{workspace_id}/prompt`    | `prompt` set; `prompt_file` and `prompt_package` to defaults, since a route-set prompt has always won |
| `DELETE /api/host/workspaces/{workspace_id}/prompt` | All three to defaults: no standing prompt, `source` answers `default`                                 |

The `workspace_prompt` table is deprecated, read only by the seed ([runner-store.md](./runner-store.md) §Seeding), and
dropped later (`engineering:deprecations-outlive-epic`).

## Flows

### Adding a runner

```mermaid
sequenceDiagram
    participant Op as Operator (CLI / page)
    participant H as Host API
    participant Hub
    participant S as Host store
    participant T as Host tick
    Op->>H: POST /api/host/runners
    H->>H: validate settings as a write
    alt invalid
        H-->>Op: 422 settings_invalid (nothing minted)
    end
    H->>Hub: add runner, host token (as runner/cli/runners.py does today)
    Hub-->>H: runner_id, hub token
    Note over H,S: crash here: minted at the hub, nothing on the host
    H->>S: one transaction: hosted_runners, `added` fact, runner_identity,<br/>sealed token, runner_settings rev 1, entries (name, projects, settings), change op create
    H-->>Op: view, state "starting"
    T->>S: next host tick lists hosted_runners
    T->>T: composition.add_runner; app.state.runners; tick, trace-sweep, spawner drivers;<br/>resume.on_startup() and takeover recovery, as a boot does
    T->>Hub: the runner's first tick registers it
```

- Nothing is written to `.env` or `blizzard-host.toml`, and no directory is created. The runners beside it keep ticking.
- A crash between the mint and the transaction leaves a runner the hub shows as never registered, with no row and no
  token on the host. Nothing runs under it; the operator retires it from the hub and adds again. The hub mints the id
  the rows are keyed by, so the two cannot share a transaction; `architecture/crash-correctness/runner.md` records the
  window under `bzh:crash-exemptions-runner`.

### Retiring a runner

```mermaid
sequenceDiagram
    participant Op as Operator
    participant H as Host API
    participant Hub
    participant S as Host store
    participant T as Host tick
    Op->>H: POST …/retire {force}
    H->>Hub: retire, host token
    alt holds a chunk and not forced
        Hub-->>H: refused
        H-->>Op: the hub's refusal
    end
    Hub-->>H: retired
    H->>S: `retired` fact
    T->>T: next tick drains this runner alone: stop its drivers, wait out its tick,<br/>resume.on_shutdown() (under force, stops its workers as ShutdownDrain does),<br/>RunnerProcess.close_runner, remove from app.state.runners
    T->>S: delete the sealed token; identity and settings rows stay
```

A runner the hub retires another way (the hub's CLI, or with its host) is noticed as today, when the hub refuses its
token. `RetiredRunnerWatch` (`runner/hub/identity_service.py`), which today deletes the identity row and leaves the
threads running, now writes the same `retired` fact, and the same tick drains it. One retired while its host was down is
noticed and drained at the host's first tick.
