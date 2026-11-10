# `blizzard runner models`

## Interfaces

### Invocation

```text
blizzard runner models [--dir DIR] [--runner-url URL] [--graph NAME | --chunk CHUNK_ID] [--from-file] [--json]
```

```python
# blizzard.runner.cli: registered in _COMMANDS beside "status"; --dir and --runner-url as on `status`.
@click.command("models")
@click.option("--dir", "directory", default=DEFAULT_DIR, envvar=ENV_RUNNER_DIR)
@click.option("--runner-url", "runner_url", default=None, envvar=ENV_LOCAL_API_URL)
@click.option("--graph", "graph", default=None)  # mutually exclusive with --chunk: usage error, exit 2
@click.option("--chunk", "chunk_id", default=None)
@click.option("--from-file", is_flag=True)  # build from the config file even when a daemon runs
@click.option("--json", "as_json", is_flag=True)  # the hub CLI's HubCommand --json; the first --json on the runner CLI
def models(directory, runner_url, graph, chunk_id, from_file, as_json) -> None:
    # Writes nothing: not the configuration, not the runner store, not the hub.
    ...
```

| Invocation     | Pools reported                                                                        |
| -------------- | ------------------------------------------------------------------------------------- |
| no flag        | Every effective graph's declared sessions, graphs by name, sessions in authored order |
| `--graph NAME` | That graph's declared sessions                                                        |
| `--chunk ID`   | The chunk's pinned graph, merged with the chunk's defaults, plus any bare lineage     |

### Exit status

| Code | When                                                                                                                                              |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `0`  | The report printed. Findings never change the code                                                                                                |
| `1`  | The configuration failed to load, with the same message `runner host` gives                                                                       |
| `2`  | A usage error, including `--graph` with `--chunk`                                                                                                 |
| `3`  | `--graph` or `--chunk` was given and its pools could not be reported: hub unreachable or unsupported, or the name unknown. The tables still print |

### JSON form (`schema: 1`)

`--json` prints one JSON document to stdout and nothing else. Every layer's value appears beside the effective one. A
new key keeps `schema` at `1`; renaming or removing a key raises it. The [sample](../../assets/model-resolution.json) is
the whole shape on one configuration.

| Key         | Holds                                                                                                                                                                                                                                           |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `schema`    | `1`                                                                                                                                                                                                                                             |
| `runner`    | `name`, `config_path`, `source` (`daemon` \| `file`), `config_loaded_at`, `config_changed_on_disk`                                                                                                                                              |
| `hub`       | `url`, `status` (`reachable` \| `unreachable` \| `unsupported` \| `not-asked`)                                                                                                                                                                  |
| `scope`     | `graph_name`, `graph_id`, `chunk_id`; each null when not named                                                                                                                                                                                  |
| `harnesses` | Per configured harness: `id`, `version`, `state`, `default`, `fallback_model` {`value`, `source`}                                                                                                                                               |
| `tiers`     | Per tier, `by_harness`: `harness`, `adapter_default`, `runner_alias`, `model`, `source`                                                                                                                                                         |
| `efforts`   | Per authored effort, `by_harness`: `harness`, `runner_alias`, `value`, `flag`, `source`                                                                                                                                                         |
| `pools`     | Per session: `graph_id`, `graph_name`, `session`, `fields` {`model`, `effort`, `harnesses`: `graph`, `chunk`, `value`, `source`}, `selection_basis`, `candidates` [{`harness`, `outcome`, `reason`, `model`, `effort`}], `selected`, `recorded` |
| `notes`     | The `!` findings: `kind` (`unmapped-tier` \| `dropped-effort` \| `recorded-differs` \| `stale-daemon-config`) and what it names                                                                                                                 |

Every `source` is a term of the [provenance vocabulary](./resolution.md#provenance-vocabulary); every `outcome` and
`reason` is a term of [harness selection](./resolution.md#harness-selection); `pools[].fields` is the hub's
[`MergedSessionView`](./hub-sessions.md#interfaces).

### Rows

| Section   | Rows                                                                                                                                                                                                      |
| --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Harnesses | Every harness enabled in the configuration, including an unavailable one: version, state (`available` \| `unavailable` \| `unhealthy` \| `not-probed`), whether it is the default, and its fallback model |
| Tiers     | One per tier and harness, over `blizzard:frontier`, `blizzard:advanced`, `blizzard:basic`, every tier keyed in any of this runner's model alias tables, and every tier authored in a reported pool        |
| Efforts   | One per effort and harness, over `low`, `medium`, `high`, `max`, every key in any of this runner's effort alias tables, and every effort authored in a reported pool                                      |
| Pools     | One block per session: the three effective fields with their sources, one line per candidate in set order with outcome, model, and effort, then the recorded pool head where one exists                   |

### Terminal rendering

```text
<header: runner, config, source, hub, scope>
HARNESSES
TIERS
EFFORT
SESSION POOLS
```

- Plain `click.echo`, fixed-width columns, no color needed to read it.
- Fits 120 columns; a long model name widens its column and is never truncated.
- A missing value renders `—`. Sources render as words: `runner alias`, `adapter default`, `chunk default`.
- A line starting `!` follows the row it concerns, for exactly four findings: a tier a harness cannot map, a dropped
  effort, a recorded pool head that differs from the next fresh mint, and a daemon running configuration older than the
  file. Nothing else gets a `!`.
- The [worked sample](../../assets/model-resolution.txt) is the reference layout.

## Flows

One report builder serves both paths; they differ only in where configuration and harness health come from.

```mermaid
sequenceDiagram
    participant CLI as runner models (CLI)
    participant D as runner daemon
    participant H as hub
    participant S as runner store
    alt daemon reachable (found as `runner status` finds it), no --from-file
        CLI->>D: local API read (report)
        Note over D: config, bindings, harness health loaded at startup
        Note over D: config file differs from what was loaded: header line + config_changed_on_disk + stale-daemon-config
    else no daemon, or --from-file
        Note over CLI: load the config as `runner tick` does, build its bindings, run no selftest
        Note over CLI: every configured harness is not-probed; selection treats each as available and says so
    end
    Note over D,CLI: the builder runs in whichever process holds the config
    opt pools in scope
        D->>H: GET /api/fleet/sessions[?graph] or /api/fleet/chunks/{id}/sessions (runner token from config)
        alt 200
            H-->>D: list[MergedSessionView]
        else unreachable, or 404 from an older hub
            Note over D: hub.status unreachable | unsupported; pools replaced by one line naming which; tables still print
        end
    end
    opt --chunk
        D->>S: pool_head(chunk_id, session) per session (read only)
    end
    D-->>CLI: report (schema 1)
    Note over CLI: render the terminal table, or print the JSON document
```

## Open questions

- The daemon's local API route for the report, its path and response model, is not named. Engineering answers.
- `runner status` also takes `--runner`, and the offline loader (`load_runner(directory, runner=...)`) needs a runner
  name on a host of several. Whether `models` takes `--runner` is not settled. Engineering answers.
- The runner's configuration now lives in `blizzard-host.toml`, with `blizzard-runner.toml` as the pre-host layout.
  Which file `config_path` and the on-disk change check name on a host is not settled. Engineering answers.
