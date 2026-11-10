# Lifetimes in the registry

Every key's lifetime is [settings.md](./settings.md).

## Domain

```python
# foundation/config_keys.py                                                   product:declared-lifetime
@dataclass(frozen=True)
class ConfigKey:
    path: str
    type: str
    required: bool = False
    default: object = None
    deprecated: bool = False
    replaced_by: str | None = None
    flag: str | None = None
    # Required on every key that holds a value, OPEN_TABLE included (`hub_tokens`, `models.aliases`: the whole value
    # is one setting). Absent on TABLE and ARRAY_OF_TABLES keys (`tracing`, `auth`, `runner`, `workspace`,
    # `workspace[].repo`), which only group children that carry their own.
    # A deprecated key keeps the lifetime of the key replacing it. One read only to be refused and whose
    # replacement is a record kind, not a key, is `BOOT`: the hub's `work_source[]`, `replaced_by` the work-source
    # records.
    lifetime: Lifetime | None = None
    # The environment variable that supplies the value and overrides the file. Legal only on a BOOT key.
    env: str | None = None                                                    # product:boot-lifetime


@dataclass(frozen=True)
class ConfigRegistry:
    def __post_init__(self) -> None:
        for key in self.keys:
            groups = key.type in (TABLE, ARRAY_OF_TABLES)
            if groups and key.lifetime is not None:
                raise ValueError(f"{key.path}: a grouping key carries no lifetime")
            if not groups and key.lifetime is None:
                raise ValueError(f"{key.path}: a value key declares its lifetime")
            if key.env is not None and key.lifetime is not Lifetime.BOOT:
                raise ValueError(f"{key.path}: only a boot key is supplied by an environment variable")
        ...
```

## Interfaces

### Environment variables

The one list of variables that supply a setting:

| Key          | Daemon | `env`           |
| ------------ | ------ | --------------- |
| `host_token` | runner | `BZ_HOST_TOKEN` |
| `host`       | hub    | `BZ_HUB_HOST`   |
| `port`       | hub    | `BZ_HUB_PORT`   |
| `db_url`     | hub    | `BZ_HUB_DB_URL` |

`BZ_RUNNER_TICK_SECONDS` supplies `tick_seconds` only through the seed until it is removed, and is declared on no key
(`product:tick-interval-setting`). Variables declared through no `env`:

| Variable                                                                                              | Why not                                                                                                                                                                                            |
| ----------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| The one named by `session_secret_env`, `egress.path_key_env`, an OAuth provider's `client_secret_env` | The naming key is an ordinary `boot` key; the variable it names is read at boot and supplies a secret, not the key                                                                                 |
| `BZ_HUB_TOKEN_<NAME>`                                                                                 | Keeps its `EnvFile` declaration (`runner/config_keys.py`), the family supplying `hub_tokens`; both deprecated, read only by the token import ([runner-store.md](./runner-store.md) §Sealed tokens) |
| `BZ_RUNNER_DIR` (`runner/cli/env.py`)                                                                 | The CLI's runtime root; supplies no key                                                                                                                                                            |
| `BZ_HUB_SECRET_KEY` (`hub/secrets/internal/env_keys.py`), `BZ_HOST_SECRET_KEY`                        | Sealing keys; supply no key ([runner-store.md](./runner-store.md) §Sealed tokens)                                                                                                                  |
| OpenTelemetry's own variables                                                                         | Not settings                                                                                                                                                                                       |

### Contract documents

`tools/config_contract.py` writes two fields on every entry of `keys[]` in `contracts/config/hub.json` and
`contracts/config/host.json`; `contracts/config/README.md` documents them and the four values.

```json
{ "path": "throttle.budget.usd", "type": "number", "default": null, "lifetime": "pass", "env": null }
```

A lifetime changed without a regenerated document turns the `contract drift` step red; `blizzard:config-contract` holds
the registries to the documents. The two fields are additive (`bzh:released-surface-additive`).

```python
# blizzard:compat, beside the removed-key check; docs/versioning.md §What counts as breaking gains the row
ORDER = (Lifetime.PASS, Lifetime.SESSION, Lifetime.BOOT, Lifetime.IMMUTABLE)

def lifetime_breaks(candidate: ContractKey, released: ContractKey) -> bool:
    # Later in ORDER narrows what an operator changes without a restart: a break. Earlier is additive.
    return ORDER.index(candidate.lifetime) > ORDER.index(released.lifetime)
```

### Reading entries

`ConfigView` keeps serving `boot` keys from the file. An entry is read only through its daemon's settings reader:
[runner-store.md](./runner-store.md) §The reader, [hub.md](./hub.md) §Reading per pass. Once the last of a daemon's
`pass` and `session` keys is an entry, a read of one through `ConfigView` outside the seed is a
`bzh:config-key-declared` finding (`engineering:context-rules`).

## Flows

### Seed

Run at every daemon start, one entry at a time, before anything reads an entry (`product:runner-file-seeds`,
`product:hub-file-seeds`):

```python
# The rule per entry, for both daemons' seeds.
def seed_entry(store: SettingsStore, key: ConfigKey, file: ConfigView) -> None:
    if store.has_entry(key.path):        # an entry `at_default` counts: an unset is never re-seeded
        return                           # the store's value stands; the file's is ignored, with no warning
    if not file.states(key.path):
        return                           # resolves to the registry default; nothing is written
    # One key's whole value: a list, an alias table, or `throttle.pace.taper[]` is one entry, never seeded element
    # by element. Written through the same validation as an API write, with change-log door and actor `seed`; a
    # refusal fails the start with the message the same value fails a boot with today.
    store.write({key.path: file.value(key.path)}, door="seed", actor="seed")
```

No environment variable seeds an entry but `BZ_RUNNER_TICK_SECONDS`, and that only where neither store nor file holds
`tick_seconds`. A per-runner entry is scoped to its runner: it is seeded from that runner's `[[runner]]` table the first
time the runner starts, and one runner's table never seeds another's. The runner's seed order is
[runner-store.md](./runner-store.md) §Seeding; the hub's runs in `blizzard hub serve` ([hub.md](./hub.md) §Data).

### Evaluated at the start of a pass

The rule `bzh:settings-snapshot-per-pass` states, for every pass of either daemon (`product:pass-lifetime`,
`product:every-pass-evaluates`, `product:hub-pass-settings`):

```python
# The order every pass follows; the readers are RunnerSettingsReader and HubSettingsReader.
def run_pass(reader: SettingsReader, built: BuiltCache, work: Callable[[Snapshot, Built], None]) -> None:
    # 1. Before any other work: one indexed read of the revisions of every settings record the pass reads
    #    (bzh:live-set-read); entries are read only where a revision moved since the last snapshot built.
    snapshot = reader.snapshot()
    # 2. Anything built from an entry (a hub client's timeout, an alias table, a sampler) is kept under the
    #    revisions it was built from, reused while they hold, rebuilt when they move (bzh:config-read-on-use).
    objects = built.at(snapshot.revisions)
    # 3. The pass works from this snapshot to its end and reads no entry again; a write committing meanwhile
    #    reaches the next pass. A step inside the pass takes no snapshot of its own.
    # 4. A worker the pass launches is handed the snapshot's `session` keys and keeps them for its session.
    work(snapshot, objects)
```

A hub request is a pass: it takes its snapshot on arrival. Neither daemon watches its store or pushes a write into a
running object; a later pass finding a moved revision is the only way a change arrives
(`decision:config/runtime/pull-per-pass`).
