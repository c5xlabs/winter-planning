# Service provider contract

The capability registry, the `service` slot at version `v1`, and everything that crosses between the binary and a
service provider. External providers keep the reference tool's contract unchanged; the tmux, docker, and process-compose
providers are part of the binary (`decision:running-environments/service-orchestration/builtin-execution-placement`).
Reference locations are paths under `tools/winter-cli/src/winter_cli/` of the `winter-workspace` repository at
`b57b020`. Variable groups named here are defined in the
[invocation contract](../../../agent-ready-operation/executable-extensions/spec/invocation-contract.md); the manifest
keys are the [extension manifest](../../../agent-ready-operation/extension-management/spec/extension-manifest.md).

This is a contract toward other programs: argument vectors, environment, the manifest file, and the documents read back
are fixed exactly. Messages the binary writes about a provider are its own wording (`decision:functional-parity`).

## System

```text
slot              service                      the only slot
supported spec    v1                           modules/capability/specs/service-v1.toml
external provider an adopted extension whose manifest sets [provides] service (or the legacy orchestrate_services)
built-in provider tmux | docker | process-compose, compiled into the binary; no Python, no extension checkout needed
```

How a built-in and an external provider of the same backend coexist — naming, precedence, whether a built-in is bound
without a declaration — is **not specified**:
`decision:running-environments/service-orchestration/builtin-external-coexistence`. Everything below binds external
providers, and binds built-ins wherever it describes behaviour the caller of `service` can observe.

## Flows

### Binding

```text
# product:provider-binding, decision:running-environments/service-orchestration/provider-activation
# reference: modules/capability/capability_registry_service.py:95-343
candidates = adopted extensions providing the slot

[capabilities] service set (explicit), each name in order:
    not a registered extension, or not providing the slot   -> invalid
    [implements] service set and not a supported version    -> incompatible; the failure carries the slot, the
                                                               version declared, and the versions supported
    otherwise                                               -> bound, kind explicit
not set:
    0 candidates                                            -> unbound
    1 candidate                                             -> bound, kind implicit
    2 or more                                               -> all bound, sorted by name
```

An absent `[implements]` entry is compatible. `capabilities` reports the binding; a broken slot there still exits 0.

### Override

```text
# reference: modules/service/orchestrator_resolver.py:32,82
precedence:  --service-orchestrator <v>   >   WINTER_SERVICE_ORCHESTRATOR=<v>   >   the binding above
<v> contains a path separator, or names an existing directory  -> that directory's winter-ext.toml is read directly
otherwise                                                      -> looked up as a registered extension name
effective override -> one notice on stderr naming <v>, for every `service` command, `catalog` included
```

### Fan-out

`up`, `down`, and `status` are dispatched per cell; `restart` is routed differently, below.

```text
# reference: modules/service/service_dispatch_service.py:112-174; service_fan_out_service.py:146-239;
#            service_status_matrix_service.py:129-218,278-302,391-501; provider_invocation.py:95-109
a pattern is <scope>/<service>, or a bare <scope>, which selects every service of that scope
scope is `workspace` or a configured env name

cells       one per (provider, scope) the patterns select; env scopes sorted by name, providers in bound order
            within a scope, workspace cells last
            a scope is selected when the scope segment of some pattern matches it, as a case-sensitive glob over
            the scope name; `workspace` is matched only by the literal `workspace`, never by a glob
            `status` with no pattern selects every scope
            one provider:  every selected scope gives it a cell; `describe` is not called
            two or more:   `describe` is called on each; a provider gets a cell for an env scope only when it owns
                           env services, and for `workspace` only when it owns workspace services; when every
                           pattern names a service, also only when one of its services matches a pattern
filters     the service filters of a cell are the distinct service segments of the patterns whose scope segment
            matches the cell's scope, in the order first given
            a bare pattern that matches the scope removes every filter for it
```

Calls per cell. Each call carries exactly one positional.

| Service filters of the cell | `up`, `down`                                                   | `status`                                                                    |
| --------------------------- | -------------------------------------------------------------- | --------------------------------------------------------------------------- |
| none                        | one call, `<scope>`                                            | one call, `<scope>/*`                                                       |
| one                         | one call, `<scope>/<service>`                                  | one call, `<scope>/<service>`                                               |
| two or more                 | one call per filter, in filter order, each `<scope>/<service>` | one call, `<scope>/*`; the merged document is then narrowed to the patterns |

So `down alpha/db alpha/api` makes two calls, `down alpha/db` and `down alpha/api`, and never `down alpha`.

Per verb:

| Verb     | Scope environment                                    | Order and failure                                                                                          | Exit                                            |
| -------- | ---------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- | ----------------------------------------------- |
| `up`     | command entries are executed, at most once per scope | calls run one at a time in cell order; the first non-zero code ends the fan-out, and no later call is made | that first non-zero code, else `0`              |
| `down`   | command entries are not executed; they are masked    | calls run one at a time in cell order; every call is made whatever the earlier ones returned               | the first non-zero code, else `0`               |
| `status` | command entries are not executed; they are masked    | cells are called concurrently; every call is made                                                          | the first non-zero code in cell order, else `0` |

The scope environment — scope trio, workspace port, bands — is computed once per scope within one dispatch and reused by
every call for that scope, so the per-filter calls of one cell and the cells of several providers share it.

```text
no cell selected, `up` or `down`:
    one diagnostic on stderr naming the patterns; exit 1; no `up` or `down` call is made
```

### Restart routing

```text
# reference: modules/service/service_dispatch_service.py:176-301; provider_invocation.py:112-197
restart <patterns>: no cells; a provider is called at most once, with every pattern routed to it as positionals,
                    each exactly as given
scope check, per pattern:
    its scope segment — the whole token when the pattern is bare — is `workspace`, or `*`, or matches a configured
    env name as a case-sensitive glob in either direction
one provider:
    any pattern fails the scope check   -> one diagnostic on stderr naming each such pattern; exit 1; no call
    otherwise                           -> one call with all patterns in the given order; `describe` is not called
                                           exit: the provider's code
two or more:
    `describe` is called on each to learn which provider owns which service
    a pattern matches a service segment-wise, scope against scope and service against service, as a case-sensitive
    glob in either direction; `workspace` matches only exactly; a bare pattern is <scope>/*
    each provider is routed the patterns that match a service it owns, in the given order, each once
    a pattern that fails the scope check, or a <scope>/<service> pattern that matches no known service
                                        -> one diagnostic on stderr naming each such pattern; exit 1; no
                                           `restart` call
    a provider routed no pattern is not called
    every other provider is called, in bound order, whatever the earlier ones returned
                                           exit: the first non-zero code, else 0
```

### Implicit workspace start

```text
# reference: modules/service/handler.py:75-97
service up <patterns>:
    no pattern is `workspace` or starts with `workspace/`
        -> first dispatch `up workspace`, best effort; then dispatch the given patterns
           both dispatches are attempted whatever the first returns
           exit: the first non-zero code, the workspace dispatch's before the patterns'
    some pattern is `workspace` or starts with `workspace/`
        -> no implicit dispatch
--wait runs only when every dispatch returned 0, and gates only the given patterns, never the implicit
workspace dispatch
```

`service start` behaves the same, being the same command.

## Interfaces

### Invocation

```text
# reference: core/internal/local_subprocess_runner.py:193-218,254-281
argv:   <entrypoint, absolute> <action> [positionals…]
cwd:    workspace root
stdin:  inherited from the binary; a provider can read the terminal
env:    the binary's environment, plus the base five, plus the table below; for an action whose row carries the scope
        trio, the binary's environment first loses the names of the remove list of the Launch environment of
        ../../scope-environment/spec/consumers.md, so a caller's stale scope identity never reaches the provider
        (decision:running-environments/scope-environment/inherited-environment)
        GIT_TERMINAL_PROMPT is not set by the binary; a value already in its environment is inherited
```

| Action     | Positionals                             | Environment beyond the base five                                                                                  |
| ---------- | --------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `up`       | one, per Fan-out; patterns are required | scope trio, workspace port, bands; `WINTER_SERVICE_TIMEOUT`; `WINTER_SERVICE_MANIFEST` when any definition exists |
| `down`     | one, per Fan-out; patterns are required | scope trio, workspace port, bands with command entries masked and not run                                         |
| `status`   | one, per Fan-out; patterns are optional | scope trio, workspace port, bands with command entries masked and not run                                         |
| `restart`  | patterns, required                      | none                                                                                                              |
| `logs`     | patterns, required                      | none                                                                                                              |
| `describe` | none                                    | none                                                                                                              |
| `catalog`  | none                                    | none                                                                                                              |

`WINTER_SERVICE_TIMEOUT` is seconds as a decimal string that parses as a float, default `120.0`, set on `up` whether or
not `--wait` is given. `down` receives neither it nor `WINTER_SERVICE_MANIFEST`.

`logs` argument vector:
`logs <pattern…> --tail <N|all> [--since <rfc3339>] [--until <rfc3339>] [--follow] [--timestamps]`.

| Provider exit | Meaning                                |
| ------------- | -------------------------------------- |
| `0`           | Success.                               |
| `2`           | Unknown action.                        |
| `3`           | Recognised action, not implemented.    |
| other         | Failure; passed through to the caller. |

A provider entrypoint that cannot be executed — missing, or not executable — yields `126` in place of a provider code on
`up`, `down`, `restart`, and any other action dispatched to the one resolved provider
(`core/internal/local_subprocess_runner.py:271-281`). It is then treated as that call's code: it ends an `up` fan-out
and is the candidate exit of a `down` or `restart`.

### Service manifest file

Written on `service up` only, when the workspace or any adopted extension declares a service
(`modules/service/service_manifest_collector.py:37,92-105`, `ext_service_manifest.py:266-300`). Workspace definitions
come first, then each extension's. An extension whose manifest fails to load contributes none and is skipped with a
warning ([extension manifest](../../../agent-ready-operation/extension-management/spec/extension-manifest.md), Failure
handling).

```text
path:      <system temp dir>/winter_service_manifest_<random>.toml, passed as WINTER_SERVICE_MANIFEST; never deleted
per definition, in this order; name, scope, source always; the last four only when non-empty:
    [[service]]
    name    = <q>
    scope   = <q>
    source  = <q>
    cmd     = <q>
    image   = <q>
    target  = <q>
    ports   = [<q>, <q>]
    <empty line>
lines are joined by one "\n"
```

```text
# decision:running-environments/service-orchestration/service-manifest-quoting
<q> is the reference tool's string repr, not TOML quoting, because existing providers parse that form:
    delimiter   '…'  unless the value contains ' and no "  -> "…"
    escapes     \\  \n  \r  \t ;  the delimiter when it occurs inside ;  other non-printables as \xNN, \uNNNN, \UNNNNNNNN
    printable non-ASCII characters are written as themselves
```

`source` is `workspace` for a workspace definition and the declaring extension's resolved prefix for an extension's —
workspace entry `prefix`, else manifest `prefix`, else manifest `name`, else the repository's catalog name, which is its
declared `name` or the name derived from its URL and not the directory it is checked out in
([extension manifest](../../../agent-ready-operation/extension-management/spec/extension-manifest.md), Prefix
resolution; `modules/workspace/extension_manifest.py:318,366`). `service ext-services` emits the same value.

| Failure while collecting definitions                | Result                                                                         |
| --------------------------------------------------- | ------------------------------------------------------------------------------ |
| A service name declared twice, in one source or two | Command error carrying the name and the sources; exit 1; no provider is called |
| A malformed workspace `[[service]]` block           | Command error carrying the service entry fault; exit 1; no provider is called  |

Whether a provider can act on a contributed service after the `up` that passed the file is the provider's own state:
`decision:running-environments/service-orchestration/declaration-continuity`.

### Documents read from stdout

`status`:

```json
{"envs": [{"env": "alpha", "session": "winter-alpha", "port_base": 4020,
           "services": [{"name": "api", "state": "running", "health": "healthy", "ports": [4021],
                         "handle": "winter-alpha:api", "log_path": "/abs/api.log", "since": "2026-01-01T00:00:00Z"}]}]}
```

| Field                          | Type                                  | When absent or wrong                         |
| ------------------------------ | ------------------------------------- | -------------------------------------------- |
| document                       | object                                | error: invalid JSON, or not an object        |
| `envs`                         | array                                 | error: missing or not an array               |
| `envs[].env`                   | string                                | `""` (`modules/service/status_parser.py:78`) |
| `envs[].session`               | string or null                        | null                                         |
| `envs[].port_base`             | integer or null                       | null                                         |
| `envs[].services[].name`       | string                                | `""` (`status_parser.py:107`)                |
| `…state`                       | `running` \| `stopped` \| `unknown`   | `unknown`                                    |
| `…health`                      | `healthy` \| `unhealthy` \| `unknown` | `unknown`                                    |
| `…ports`                       | array of integer                      | non-integers dropped                         |
| `…handle`, `log_path`, `since` | string or null                        | null                                         |

`describe`: `{"services": ["<name>", …]}`; the only errors are invalid JSON and a non-object.

`catalog`: `{"services": ["workspace/<name>", "*/<name>", …]}`; a provider whose `catalog` call fails is omitted with no
message.

Log lines, one JSON object per line:

| Field | Type   | Rule                                                           |
| ----- | ------ | -------------------------------------------------------------- |
| `ts`  | string | Optional.                                                      |
| `env` | string | With patterns given, a line lacking `env` or `svc` is dropped. |
| `svc` | string | Same.                                                          |
| `msg` | string | A line that is not JSON is kept whole as `msg`.                |

### Readiness

```text
# product:truthful-service-readiness, decision:running-environments/service-orchestration/readiness-enforcement
# reference: modules/service/service_readiness_service.py; modules/service/handler.py:86-97
service up --wait [--timeout SECONDS]:        default 120.0
    after every up dispatch returned 0, collect `status` for the given patterns, the same way `service status` does
    a poll is READY when no service in the merged document has health == "unhealthy"
        "healthy" and "unknown" both pass
        no parseable status document from any provider passes
        a document with no services passes
    READY                              -> exit 0
    not READY and the timeout has elapsed since the first poll began
                                       -> exit 1; one diagnostic on stderr carrying the timeout, the patterns,
                                          and each still-unhealthy service as <env>/<service>
    not READY otherwise                -> wait 1.0 s, poll again
    interrupt while polling            -> exit 130
at least one poll always runs, so `--timeout 0` still passes when nothing is unhealthy
no provision state is recorded or consulted; nothing gates `up` on an earlier `provision`
```

### Conformance

`ext verify EXTENSIONS...` runs these against each operand's real entrypoint; an operand is an extension name or a path
to an extension directory, and there is no slot operand
([command surface](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md)):

| Check                 | Passes when                                            |
| --------------------- | ------------------------------------------------------ |
| `accepts-action`      | Each action word exits with a code other than `2`.     |
| `refuses-unknown`     | An unknown action word exits non-zero.                 |
| `forwards-params`     | The entrypoint receives the argument vector unchanged. |
| `emits-describe-json` | `describe` writes a JSON object.                       |

## Open questions

- How a built-in and an external provider of the same backend coexist, and so which of this contract's process-level
  elements — the manifest file, the environment, the argument vector — a built-in provider still produces when no
  process is spawned, is `decision:running-environments/service-orchestration/builtin-external-coexistence`. The product
  owner answers when `epic:running-environments/service-orchestration` is refined; engineering completes the built-in
  rows then.
- The exact inputs each conformance check sends are not traced from the reference tool. Engineering records them from
  `modules/ext/verify_service.py` before `ext verify` is specified.
