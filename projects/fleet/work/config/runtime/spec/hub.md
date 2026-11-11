# The hub's settings

Which keys move is [settings.md](./settings.md) §Hub.

## System

Every configured record the hub has today is a tenant's: `tenant_id` on its table, change rows in `config_changes`
(`hub/store/schema.py`, `tenant_id` not null), revisions read through `ScopedConnections`, the tenant's `LiveConfig`
cache (`hub/composition.py`), tenant-bound routes. The hub's operational settings are one hub-wide record beside them,
in none of them (`product:hub-settings-record`). The hub runs as one process, so a write's wake of a waiting sweep is
in-process.

## Domain

```python
# hub/auth/permissions.py, beside EXPORT_ADMIN
class Permission(StrEnum):
    ...
    # Operate the hub-wide settings every tenant shares. Held by a hub administrator (hub_admin_facts) and, under
    # auth.mode = none, the implicit operator. No tenant role holds it: a tenant's superuser cannot slow another
    # tenant's sweeps.
    SETTINGS_ADMIN = "settings:admin"
```

`bzh:configured-record` gains the hub settings record as a kind with no retire or enable (there is always exactly one),
keyed by the entry rather than a field; `bzh:config-read-on-use`'s scope gains it beside the tenant's records.

## Data

```python
# hub/store/schema.py — all three in GLOBAL_TABLES, one additive revision applied by `blizzard hub migrate`
# (bzh:manual-migrations); contracts/store/hub.json regenerated.

hub_settings = Table(                                    # one row
    "hub_settings", metadata,
    Column("revision", Integer, nullable=False),
    Column("updated_at", UtcDateTime, nullable=False),
)

hub_setting_entries = Table(
    "hub_setting_entries", metadata,
    Column("key", String, primary_key=True),
    Column("value", Text, nullable=True),                # JSON
    Column("at_default", Boolean, nullable=False),
    Column("revision", Integer, nullable=False),
)

hub_setting_changes = Table(                             # append-only
    "hub_setting_changes", metadata,
    Column("id", Integer, primary_key=True, autoincrement=True),
    Column("recorded_at", UtcDateTime, nullable=False),
    Column("revision", Integer, nullable=False),
    Column("op", String, nullable=False),                # "seed" | "edit"
    Column("door", String, nullable=False),
    Column("actor", String, nullable=False),
    Column("apply_id", String, nullable=True),
    Column("diff", Text, nullable=False),                # JSON [{key, old, new}]
)
```

- A write is the shared one ([lifetimes.md](./lifetimes.md) §The shared settings record), as the runner's is, over these
  tables: `If-Match` against `revision`, the whole resolved configuration validated through the section parsers
  `HubConfig.load` runs over the file (`hub/config.py`), factored out to run over resolved settings; one revision and
  one change row per committed write; `at_default` for an unset.
- The seed ([lifetimes.md](./lifetimes.md) §Seed) runs in `blizzard hub serve` after its store check and before
  `build_services`.

## Interfaces

Every route takes `require_hub(Permission.SETTINGS_ADMIN)` (`hub/api/auth_session.py`), reads included
(`decision:config/runtime/hub-settings-admin-reads`). Routes are described in `openapi/hub.openapi.json` and reach the
board through its generated client; the CLI verbs are in `contracts/cli/`. None is on the fleet wire.

| Method  | Path                          | CLI                                                | Answer                                                                                                                                                               |
| ------- | ----------------------------- | -------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `GET`   | `/api/admin/settings`         | `blizzard hub settings show [KEY]`                 | `HubSettingsView`: the runner's view shape ([runner-api.md](./runner-api.md) §Views), only `pass` keys, no `in_force`: every pass and request takes its own snapshot |
| `PATCH` | `/api/admin/settings`         | `blizzard hub settings set KEY VALUE`, `unset KEY` | `set` and `unset` with `If-Match`, refused as the runner's write is                                                                                                  |
| `GET`   | `/api/admin/settings/changes` | `blizzard hub settings changes`                    | The change log, newest first, paginated                                                                                                                              |
| `POST`  | `/api/admin/settings/apply`   | `blizzard hub settings apply FILE [--dry-run]`     | §Apply                                                                                                                                                               |
| `GET`   | `/api/admin/settings/export`  | `blizzard hub settings export`                     | The apply document, as the store stands                                                                                                                              |

The hub already serves a tenant's settings at `/api/settings` (`hub/api/settings.py`, `wire/urgency.py::SettingsView`).
The hub-wide record's wire models are `HubSettingsView`, `HubSettingRow`, and `HubSettingsWrite`, so the two never share
a name in `openapi/hub.openapi.json` or the board's generated client.

### Apply

`product:hub-settings-document`. The hub's settings document is its own, never a section of a tenant's config document,
since a tenant's document never reaches past its tenant (`bzh:config-apply`).

| Aspect     | Rule                                                                                                                                                                                                                                               |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Shape      | `blizzard-hub.toml`'s tables and key names; states any of the hub's `pass` keys. JSON on the route and from the export                                                                                                                             |
| Decode     | `apply FILE` decodes TOML through the hub's file loader (`hub/config.py`), the one hub module `bzh:config-key-declared`'s ast-grep rule lets read TOML; no new seam, rule unchanged. Not a config document, so `bzh:config-codec` does not bind it |
| Encode     | `export` emits TOML through the emitter `HubConfig.to_toml` uses, moved to `foundation/toml_emit.py` and shared with the runner                                                                                                                    |
| Writes     | Each stated key set, nothing else touched; a `boot` key refused `setting_not_live`                                                                                                                                                                 |
| Commit     | One transaction, door `apply`, one `apply_id`; one outcome row per key; a dry run rolls it back                                                                                                                                                    |
| Round trip | The export writes every entry the store holds, so applying it back writes nothing                                                                                                                                                                  |

### Reading per pass

```python
# hub/settings/                                                              product:hub-pass-settings
class HubSettingsReader:
    def snapshot(self) -> HubSettings:
        # One indexed read of hub_settings.revision; entries re-read only when it moved; cached under the revision,
        # so a cold hub is correct.
        ...
```

Each holder takes the snapshot at its pass's start or its request's arrival:

| Held today                                                                                               | Under this epic                                                                                                                                                                                                                                |
| -------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A sweep's interval, fixed into its `Sweep` at lifespan (`hub/app.py::Sweep`, `_sweep_interval`)          | `Sweep.run` evaluates the interval before each wait from the pass before's snapshot; an interval write wakes the waiting sweep, as the runner's driver is woken ([passes.md](./passes.md) §The loop's interval). Timed or stepped stays `boot` |
| `ActionPassDriver`'s `PassBound` (`hub/composition.py`)                                                  | `action_pass_seconds` from the snapshot at each pass's start; the deadline it sets is that pass's                                                                                                                                              |
| `AnnotationReconciler.full_pass_floor`                                                                   | Computed from `annotation_interval_seconds` at each pass's start                                                                                                                                                                               |
| `TraceExportSweep`'s settings, copied at construction (`hub/domain/observability/tracing/sweep.py`)      | Read at each sweep's start; `TraceReplay` and `TraceStatusReader` read `replay_max_window` per request. Whether the sweep exists (an exporter configured) stays `boot`                                                                         |
| `EgressSweep`, `EgressWriterSettings` (`hub/domain/observability/egress/sweep.py`, `hub/composition.py`) | The `pass` keys read at each sweep's start; the writer rebuilt when `max_rows_per_file` or `min_free_bytes` moves. Directory, format, and on/off stay `boot`                                                                                   |
| `IpThrottle` (`hub/composition.py`)                                                                      | Rebuilt at the first login after its keys move, its buckets empty, which a throttle changed by hand accepts; turning it off drops it                                                                                                           |
| `TranscriptCaps`, inside each tenant's cached services                                                   | Read from the snapshot per request by the ingest service; no tenant's cached services hold them                                                                                                                                                |
| `close_forge_writes_enabled`, closed over by each tenant's work-source registry (`hub/app.py`)           | Read from the snapshot when a closer is handed out                                                                                                                                                                                             |
| `produces_mode`, `route_token_mode`, read per request from `app.state.config` (`hub/api/fleet.py`)       | Read per request from the snapshot                                                                                                                                                                                                             |

## The board

The board has no hub-administration page today; its admin shell is a tenant's. It gains **Hub settings**, shown only to
a session holding `SETTINGS_ADMIN`, built from the admin shell's record panel, revisions, and field-diff components
(`web/projects/hub/src/app/admin/config-*.ts`):

| Panel   | Shows                                                                          |
| ------- | ------------------------------------------------------------------------------ |
| Keys    | Every key of the view with value, `source`, `lifetime`; an edit per `pass` key |
| History | The change log                                                                 |

## Open questions

- `SETTINGS_ADMIN`'s string value: the spec names the member, not its value; `"settings:admin"` follows `EXPORT_ADMIN`'s
  `"export:admin"`. Engineering.
