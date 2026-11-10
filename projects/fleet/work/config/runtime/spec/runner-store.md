# The runner's settings store

## Data

One additive revision in `runner/store/migrations/versions/`, its test beside it (`bzh:colocated-migration-tests`),
tables declared in `runner/store/schema.py` with `scope` and, for a host table, `transfer` (`bzh:runner-scoped-store`,
`bzh:host-stores-confined`); `contracts/store/runner.json` regenerated. Every timestamp is stamped from the injected
clock, never a `server_default` (`bzh:injected-clock`).

```python
# --- Runner-scoped: read and written only through the runner's bound connection, never one handed a runner_id.
# Transfer and ingest carry them with the runner (decision:config/runtime/runner-scoped-settings).

runner_settings = Table(
    "runner_settings", metadata,
    Column("runner_id", String, primary_key=True),
    Column("revision", Integer, nullable=False),        # 1 at the seed; +1 per committed write
    Column("updated_at", UtcDateTime, nullable=False),
    info={"scope": "runner"},
)

runner_setting_entries = Table(
    "runner_setting_entries", metadata,
    Column("runner_id", String, nullable=False),
    Column("key", String, nullable=False),              # the key's dotted path, as the contract document spells it
    Column("value", Text, nullable=True),               # JSON: one key's whole value; null when at_default
    Column("at_default", Boolean, nullable=False),      # an unset: resolves to the running build's registry default
    Column("revision", Integer, nullable=False),        # the runner revision that last wrote it
    PrimaryKeyConstraint("runner_id", "key"),
    info={"scope": "runner"},
)

runner_setting_changes = Table(                         # append-only
    "runner_setting_changes", metadata,
    Column("id", Integer, primary_key=True, autoincrement=True),
    Column("runner_id", String, nullable=False),
    Column("recorded_at", UtcDateTime, nullable=False),
    Column("revision", Integer, nullable=False),
    Column("op", String, nullable=False),               # Op
    Column("door", String, nullable=False),             # Door
    Column("actor", String, nullable=False),
    Column("apply_id", String, nullable=True),
    Column("diff", Text, nullable=False),               # JSON [{key, old, new}] for every key the write moved
    info={"scope": "runner"},
)

# --- Host-scoped: read through HostStores. Ingest carries none of them; the destination's records stand.

host_settings = Table(                                   # one row
    "host_settings", metadata,
    Column("revision", Integer, nullable=False),
    Column("updated_at", UtcDateTime, nullable=False),
    info={"scope": "host", "transfer": HOST_TRANSFER_NONE},
)

host_setting_entries = Table(
    "host_setting_entries", metadata,
    Column("key", String, primary_key=True),
    Column("value", Text, nullable=True),               # JSON
    Column("at_default", Boolean, nullable=False),
    Column("revision", Integer, nullable=False),
    info={"scope": "host", "transfer": HOST_TRANSFER_NONE},
)

hosted_runners = Table(                                  # the host's while its newest fact is not `retired`
    "hosted_runners", metadata,
    Column("runner_id", String, primary_key=True),
    Column("added_at", UtcDateTime, nullable=False),
    Column("added_by", String, nullable=False),
    info={"scope": "host", "transfer": HOST_TRANSFER_NONE},  # ingest writes the destination's row
)

hosted_runner_facts = Table(                             # append-only
    "hosted_runner_facts", metadata,
    Column("runner_id", String, nullable=False),
    Column("recorded_at", UtcDateTime, nullable=False),
    Column("op", String, nullable=False),               # "added" | "retired"
    Column("actor", String, nullable=False),
    Column("door", String, nullable=False),
    info={"scope": "host", "transfer": HOST_TRANSFER_NONE},  # ingest writes the destination's `added` fact
)

host_workspaces = Table(
    "host_workspaces", metadata,
    Column("workspace_id", String, primary_key=True),   # immutable
    Column("revision", Integer, nullable=False),
    Column("settings", Text, nullable=False),           # JSON: every workspace[] key but id
    Column("created_at", UtcDateTime, nullable=False),
    info={"scope": "host", "transfer": HOST_TRANSFER_NONE},
)

host_workspace_facts = Table(                            # append-only lifecycle
    "host_workspace_facts", metadata,
    Column("workspace_id", String, nullable=False),
    Column("recorded_at", UtcDateTime, nullable=False),
    Column("op", String, nullable=False),               # "retired" | "enabled"
    Column("actor", String, nullable=False),
    Column("door", String, nullable=False),
    info={"scope": "host", "transfer": HOST_TRANSFER_NONE},
)

host_subscriptions = Table(
    "host_subscriptions", metadata,
    Column("slug", String, primary_key=True),           # immutable
    Column("provider", String, nullable=False),         # immutable
    Column("revision", Integer, nullable=False),
    Column("settings", Text, nullable=False),           # JSON: name, credentials_path, usage_api_base, sample_interval_seconds
    Column("created_at", UtcDateTime, nullable=False),
    info={"scope": "host", "transfer": HOST_TRANSFER_NONE},
)

host_subscription_facts = ...                            # as host_workspace_facts, keyed by slug

runner_hub_tokens = Table(
    "runner_hub_tokens", metadata,
    Column("runner_id", String, primary_key=True),
    Column("ciphertext", Text, nullable=False),         # base64 AES-256-GCM; no route, view, or change row carries it
    Column("key_id", String, nullable=False),
    Column("revision", Integer, nullable=False),
    Column("updated_at", UtcDateTime, nullable=False),
    info={"scope": "host", "transfer": HOST_TRANSFER_NONE},  # ingest re-seals the moving runner's token
)

host_setting_changes = ...                               # as runner_setting_changes without runner_id, plus:
#   Column("record_kind", String, nullable=False)        # "host" | "workspace" | "subscription"
#   Column("record_key", String, nullable=True)          # workspace_id or slug

# --- lease_spawns gains, written with each spawn (passes.md §Sessions); null on a spawn recorded before the revision.
#   Column("host_settings_revision", Integer, nullable=True)
#   Column("runner_settings_revision", Integer, nullable=True)
#   Column("transcripts_root", Text, nullable=True)
# The revisions stay on the runner: no fact the runner sends the hub carries them (product:settings-in-force).
```

```python
Op = Literal["seed", "edit", "create", "retire", "enable"]
Door = Literal["seed", "cli", "page", "api", "apply"]   # set by the server from the route; a client claims only cli or page
# actor: the session's user; "operator" over the unix socket; "seed" for a seed
```

- An add and a seed write the runner's `runner_identity` row in the transaction that writes its settings, so
  `runner:rows-name-registered-runner` (`runner/store/internal/invariants.py`) holds after every crash point. A
  retirement keeps the identity row with the settings rows it describes.
- The host tick lists runners from `hosted_runners` and reads each runner's settings through the bound connection it
  builds for that runner. A runner's binding is made from `hosted_runners`, never once from the host file's `id` nor
  reseeded from the file (`engineering:context-rules`).

## Interfaces

### A write

Every write (an API write, a seed, an apply, an add, a retire) is one store transaction:

```python
def write(conn, record: SettingsRecord, change: Change, *, if_match: int | None, door: Door, actor: str) -> View:
    current = record.revision(conn)                      # an apply and a seed read it inside this transaction
    if if_match != current:
        raise Refused(409, "revision_conflict")
    resolved = record.resolve(conn, boot=file_boot_keys, overlay=change)
    errors = runner.host_config.parse(resolved, ...).errors  # the validation a boot applies to the file
    if errors:
        raise Refused(422, "settings_invalid", errors)   # every failing key named; nothing committed
    moved = record.write_entries(conn, change)           # unset → at_default=True, value=None
    if moved:                                            # a write that moves nothing increments and appends nothing
        record.bump_revision(conn)                       # once per write
        record.append_change(conn, diff=moved, op=change.op, door=door, actor=actor)
    return record.view(conn)
```

A write has no window of its own. `runner add` crosses the hub and the store; its window is
[host-records.md](./host-records.md) §Flows.

### The reader

```python
# runner/settings/
class RunnerSettingsReader:
    # Built over one runner's bound connection and the host's HostStores, so never handed a runner_id.
    # The host tick holds one per hosted runner.

    def revisions(self) -> Revisions:
        # The host revision, the runner's, and every active workspace and subscription record's, in one indexed read.
        ...

    def resolve(self, revisions: Revisions) -> RunnerConfig:
        # The file's boot keys, held from boot, overlaid with the entries at these revisions; cached under them.
        ...
```

- `HostConfig.load` (`runner/host_config.py`) still loads the file at boot and from then on supplies only the `boot`
  keys and the seed.
- `RunnerConfig` keeps its shape; every consumer reads one the reader resolved and never learns where a value came from.
- The app's federation, role policy, and public origins come from the host's own `boot` keys, not `runners[0].config`
  (`runner/cli/runtime.py`, `runner/app.py`), so a host whose first runner is retired at runtime keeps serving.

### Sealed tokens

A runner's hub token moves from `.env`, keyed by name, to `runner_hub_tokens`, keyed by id
(`product:runners-come-and-go`).

```python
# foundation/secrets/ — moved from hub/secrets/internal/{aes_gcm,key_material,env_keys,directory_keys}.py and
# hub/secrets/rotation.py; the hub keeps its keys, ciphertexts, and behavior.

def seal_hub_token(token: str, *, runner_id: str, revision: int, key: KeyMaterial) -> bytes:
    # AES-256-GCM; runner_id and revision are the associated data, so a ciphertext copied to another row
    # does not open.
    ...

# The host's key: BZ_HOST_SECRET_KEY, with BZ_HOST_SECRET_KEY_PREVIOUS readable during a rotation, supplied as the
# hub's key is. Where neither is set, a key directory under the host's runtime root, minted at the first start that
# needs it, owner-only, declared a durable path of the host registry.
```

| Event                                                        | Token                                                                                                                                              |
| ------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Host builds a runner's hub clients, or the token row changes | Opened; never at a pass. `HubTokenHolder` (`runner/hub/held_token.py`), which re-reads `.env` while identity is in doubt, re-reads the row instead |
| Add                                                          | The token the hub returned is sealed in the add's transaction                                                                                      |
| Retire                                                       | Ciphertext deleted; the row's history stays in the change log (§Open questions)                                                                    |
| Seed step 6                                                  | Imported from `BZ_HUB_TOKEN_<NAME>`, then `[hub_tokens]`, then `.env` (`runner/host_config.py::resolve_token`)                                     |

Deprecated and named in the upgrade note, readable until a later release removes them
(`engineering:deprecations-outlive-epic`): `BZ_HUB_TOKEN_<NAME>`, `[hub_tokens]`, and `.env`'s runner variables, read
only by the seed; `runners/<name>/` and its durable path, which `add` and `ingest` stop creating and nothing reads.
`runner add` stops writing `.env`; `runner retire` stops editing it.

## Flows

### Seeding

```python
# blizzard runner host, after ensure_current_revision (runner/runtime.py) and before any runner is built.
# Each record seeded is one write (§A write) applying lifetimes.md §Seed per entry; a record the store holds is never
# touched, and nothing in the file is ever reported as differing from the store.
def seed_host(store, file: HostConfig) -> None:
    seed_host_entries(store, file)          # 1. host-level pass/session keys; tick_seconds from BZ_RUNNER_TICK_SECONDS
                                            #    only when neither file nor store states one
    for ws in file.workspaces:              # 2. each [[workspace]] with no host_workspaces row, whole from its table.
        if not store.has_workspace(ws.id):  #    No [[workspace]] seeds none. A deprecated workspace_prompt override
            store.create_workspace(ws)      #    for it is the seeded `prompt`, with prompt_file and prompt_package at
                                            #    their defaults, since the override already won over every file prompt.
    for sub in file.subscriptions:          # 3. each [[subscription]] with no host_subscriptions row
        if not store.has_subscription(sub.slug):
            store.create_subscription(sub)
    for runner in file.runners:             # 4. hosted_runners row, `added` fact (door seed), runner_identity, entries
        if runner.id is None:               #    a scaffold not yet joined: its join seeds it
            continue
        if store.retired(runner.id):        #    retirement is final; the file cannot bring a runner back
            continue
        if not store.hosted(runner.id):
            store.add_hosted(runner)
    for runner in store.hosted_runners():   # 5. each key a hosted runner's table states with no entry
        seed_runner_entries(store, runner, file)
        if not store.has_token(runner.id):  # 6. its token as resolved today, sealed (§Sealed tokens)
            store.seal_token(runner.id, resolve_token(runner, file))
```

### No default workspace

A host has the workspaces it is given and none until then (`product:host-records`,
`decision:config/runtime/no-default-workspace`). Removed, and named breaking in the change note:

| Removed                                    | Where                                                          |
| ------------------------------------------ | -------------------------------------------------------------- |
| The `workspace-local` fallback             | `runner/config.py`, `runner/host_config.py`                    |
| `runner init`'s scaffolded `[[workspace]]` | `runner/host_config.py`, from `BZ_WORKSPACE_ROOT` and siblings |
| The `BZ_WORKSPACE_*` variables `init` read | no longer read                                                 |

A host with no workspace runs, reports none, and its runners claim nothing until one is created
([host-records.md](./host-records.md)) and a project bound to it. A legacy flat runner file still graduates to a host
file naming the workspace it always named.

## Open questions

- Whether a retirement deletes the `runner_hub_tokens` row or keeps it with its ciphertext cleared; the spec says only
  that the ciphertext is deleted and the history stays in the change log. Engineering.
