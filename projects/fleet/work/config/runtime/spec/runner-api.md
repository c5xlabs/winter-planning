# The runner's settings API

## System

Every route lives in `runner/api/`, is described in `openapi/runner.openapi.json`, reaches the local page through its
generated client (`bzh:generated-client`), and is additive to the runner's API (`bzh:released-surface-additive`).

## Interfaces

### Who may write

```python
# runner/api/wiring.py                                                       product:runner-settings-api
def require_superuser(session: RunnerSession = Depends(...)) -> RunnerSession:  # the request's human session
    # Every write below: a settings edit, an apply, a runner's add or retire, a workspace's or subscription's create,
    # edit, retire, or enable, and the workspace prompt's PUT and DELETE. Reads stay on the human lane, open to anyone
    # the local page admits. A unix-socket caller holds IMPLICIT_SESSION (runner/auth/session.py, role SUPERUSER), so
    # the CLI writes with the socket file's permission. A worker's lease token reaches only the ungated lane.
    if session.role is not Role.SUPERUSER:
        raise Refused(403, "superuser_required")
    return session
```

Pause, takeover, requeue, and selftest keep the access they have.

### Routes

`{key}` is a key's dotted path as the contract document spells it. A table key's whole value is one entry, set whole.

| Method        | Path                                              | Body / query                                  | `200` answer                                                          |
| ------------- | ------------------------------------------------- | --------------------------------------------- | --------------------------------------------------------------------- |
| `GET`         | `/api/host/settings`                              | —                                             | Host view                                                             |
| `PATCH`       | `/api/host/settings`                              | Write, `If-Match`                             | Host view                                                             |
| `GET`         | `/api/host/settings/changes`                      | `?record_kind=`, `?record_key=`; paginated    | Change rows, newest first                                             |
| `GET`         | `/runners/{runner_id}/api/settings`               | —                                             | Runner view                                                           |
| `PATCH`       | `/runners/{runner_id}/api/settings`               | Write, `If-Match`                             | Runner view                                                           |
| `GET`         | `/runners/{runner_id}/api/settings/changes`       | paginated                                     | Change rows, newest first                                             |
| `POST`        | `/api/host/settings/apply`                        | Apply document; `?dry_run=true` rolls it back | Outcome rows                                                          |
| `GET`         | `/api/host/settings/export`                       | —                                             | Apply document: every entry, every active record, every hosted runner |
| `GET`, `POST` | `/api/host/runners`                               | see [host-records.md](./host-records.md)      |                                                                       |
| `POST`        | `/api/host/runners/{runner_id}/retire`            | see [host-records.md](./host-records.md)      |                                                                       |
| six verbs     | `/api/host/workspaces`, `/api/host/subscriptions` | see [host-records.md](./host-records.md)      |                                                                       |

### Views

```json
{
  "revision": 7,
  "in_force": { "revision": 6, "since": "2026-11-05T22:14:03Z" },
  "settings": [
    { "key": "throttle.budget.usd", "value": 40.0, "source": "entry", "lifetime": "pass",
      "revision": 7, "written_at": "2026-11-05T22:15:40Z", "written_by": "operator" },
    { "key": "agents", "value": 2, "source": "default", "lifetime": "pass" }
  ]
}
```

```python
# product:settings-in-force
class SettingsView(WireModel):
    revision: int                       # the record's revision in the store
    in_force: InForce                   # the revision the most recent pass started from, and when it took hold;
                                        # behind `revision` for one tick after a write. The host view's is the host tick's.
    settings: list[SettingRow]          # every pass, session, and immutable key the registry declares for the record;
                                        # never a boot key, which is changed in the file

class SettingRow(WireModel):
    key: str
    value: object                       # no listed key holds a secret: credentials_path and opencode.auth_path are paths
                                        # to credentials; a later key that would hold one needs its own plan
    source: Literal["entry", "default"] # default: no entry, or an entry at_default
    lifetime: Lifetime
    revision: int | None
    written_at: str | None
    written_by: str | None
```

### A settings write

```json
{ "set": { "throttle.budget.usd": 40.0, "queue.strict": true }, "unset": ["agents"] }
```

Sent with `If-Match: <revision>`. `set` replaces each named entry whole; `unset` writes it `at_default`
([runner-store.md](./runner-store.md) §A write); an unnamed key is untouched. Answers the new view, or:

| Status | `code`               | When                                                                                                    |
| ------ | -------------------- | ------------------------------------------------------------------------------------------------------- |
| `403`  | `superuser_required` | The session is not the superuser                                                                        |
| `409`  | `revision_conflict`  | `If-Match` is missing or stale                                                                          |
| `409`  | `runner_retired`     | The runner's newest hosted fact is `retired`                                                            |
| `422`  | `setting_unknown`    | A key the registry does not declare for the record                                                      |
| `422`  | `setting_not_live`   | A `boot` key; the message names `blizzard-host.toml` and a restart                                      |
| `422`  | `setting_immutable`  | An `immutable` key                                                                                      |
| `422`  | `settings_invalid`   | The resolved configuration fails validation; `errors[]` names each key and its message, as a boot would |
| —      | —                    | A key in both `set` and `unset` is refused (§Open questions)                                            |

### Apply

`product:runner-settings-document`. The document is shaped like `blizzard-host.toml` (the same tables and key names) and
reconciles under the rules `bzh:config-apply` gives the hub's document:

| Document states                                        | Outcome                                                                                               |
| ------------------------------------------------------ | ----------------------------------------------------------------------------------------------------- |
| A host-level or runner key                             | Set                                                                                                   |
| A `[[workspace]]` or `[[subscription]]` with no record | Created                                                                                               |
| One with a record                                      | Edited by exactly the keys stated; enabled if retired                                                 |
| A `[[runner]]` whose `id` the host holds               | Its keys set                                                                                          |
| A `[[runner]]` whose `id` the host does not hold       | Refused `runner_unknown`: adding a runner is minted at the hub ([host-records.md](./host-records.md)) |
| A `boot` key                                           | Refused `setting_not_live`                                                                            |
| An `immutable` key                                     | Refused `setting_immutable` unless equal to the record's, which is how a document names its record    |
| Nothing for a key or record                            | Untouched; a record left out is never retired                                                         |

```python
# The hub's apply answer (wire/config.py::ConfigApplyResponse), reused
class ConfigApplyResponse(WireModel):
    dry_run: bool
    apply_id: str | None                # groups the change rows a real apply wrote; None for a dry run
    outcomes: list[ConfigApplyOutcome]  # one per entry and record the document states

class ConfigApplyOutcome(WireModel):
    kind: str                           # "host", "runner", "workspace", "subscription"
    key: str                            # the runner id, workspace id, or slug; the key for a host entry
    op: str                             # created, edited (diff names the keys it moved), enabled, unchanged, refused
    diff: list[FieldChangeView] = []
```

- Every write commits together, each with its change row, door `apply`, one shared `apply_id`, or none does. A dry run
  is the same transaction rolled back.
- The route takes and the export answers JSON. Applying an export back writes nothing.

## CLI

Verbs under `blizzard runner`, declared in `contracts/cli/runner.json`, each reaching the host through
`runner/cli/daemon.py::RunnerDaemon.reach` (the unix socket, or `--runner-url`):

| Verb                                                    | Route                                                                                          |
| ------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| `settings show [KEY] [--runner NAME] [--host]`          | The view; one key's row with `KEY`                                                             |
| `settings set KEY VALUE [--runner NAME] [--host]`       | A write; `VALUE` parsed as a TOML value, so a table or list is written as TOML inline          |
| `settings unset KEY [--runner NAME] [--host]`           | A write                                                                                        |
| `settings changes [--runner NAME] [--host]`             | The change log                                                                                 |
| `settings apply FILE [--dry-run]`                       | An apply; `FILE` is TOML decoded by `runner/host_config.py::decode_host_text`, sent as JSON    |
| `settings export`                                       | The export, emitted as TOML on stdout by `foundation/toml_emit.py` ([hub.md](./hub.md) §Apply) |
| `workspace create\|list\|show\|edit\|retire\|enable`    | The workspace routes                                                                           |
| `subscription create\|list\|show\|edit\|retire\|enable` | The subscription routes                                                                        |

- `--runner` takes a name or an id and defaults to the host's only runner, refusing to guess between several, as the
  existing operator verbs do.
- `set`, `unset`, and `edit` read the current revision and send it as `If-Match`, so two concurrent operators get a
  `409`, never a lost write.
- The host loader's decode step is the one runner module `bzh:config-key-declared`'s ast-grep rule lets read TOML; no
  new decoding seam opens and the rule is unchanged.
- `runner add` and `runner retire` keep their names ([host-records.md](./host-records.md) §Interfaces).

## The local page

`web/projects/runner` gains a **Settings** tab beside Board, Events, and Host, over the routes above:

| Panel                                            | Shows                                                                                                                                                                                                |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Host settings; Runner settings per hosted runner | Every key of the view grouped by table, with value, `source`, and a `lifetime` badge; the `in_force` line on top: "running revision 6 since 22:14 — revision 7 written, takes hold at the next tick" |
| Edit dialog per key (superuser only)             | A one-key write; each refusal error beside its key. No `boot` key                                                                                                                                    |
| Workspaces, Subscriptions                        | The records with their state, a detail with the six verbs, the record's change history                                                                                                               |
| History                                          | Change rows: door, actor, revision, and each moved key's old and new values                                                                                                                          |

A session without the superuser role sees every panel and no edit control.

## Open questions

- The status and code refusing a key named in both `set` and `unset`. Engineering.
- How an apply reports a refused entry: the spec gives a `refused` outcome row with its errors, the hub's apply answers
  a refusal with a problem response and has no `refused` op; and the `kind` values beyond the hub's. Engineering.
