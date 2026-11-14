# Winter launcher shim spec

## System

```mermaid
flowchart LR
    caller["caller argv, cwd, env"] --> cli["winter-cli"]
    cli --> platform["winter-platform"]
    platform -->|"Hub/Runner; Workspace without pin: merge root to cwd"| rc["ancestor .winterrc files"]
    platform -->|"Workspace only: pin"| workspace["nearest .winter/config.toml"]
    platform -->|"no directory selection: role default"| user["XDG_CONFIG_HOME/winter/config.toml"]
    platform -->|"read"| cache["XDG_CACHE_HOME/winter/versions/"]
    platform -.->|"execve, same PID"| component["cached winter-workspace, winter-hub, or winter-runner"]
    platform -->|"legacy command: opt-in; init: unconditional"| legacy["legacy adapter"]
    legacy -->|"read flag / init: create absent config"| workspace
    legacy -->|"when stale: sync dependencies"| uv["uv sync --project workspace/tools/winter-cli"]
    legacy -.->|"execve, same PID"| python["workspace/tools/winter-cli venv/bin/winter"]
```

| Module (`winter-platform/src/`) | Change                                                                                           |
| ------------------------------- | ------------------------------------------------------------------------------------------------ |
| `process.rs`                    | Loses its `cfg(dev-sample)` gate; `Process` and `UnixProcess` serve production dispatch          |
| `dispatch.rs`                   | New: modern version resolution, manifest validation, `ExecRequest` construction                 |
| `selection.rs`                  | New: ancestor `.winterrc` merge, Workspace pin discovery, version-string validation              |
| `legacy.rs`                     | New: opt-in Python selection, venv preparation, temporary init; injected filesystem/synchronizer |
| `cli.rs`                        | Classifies modern groups, legacy command names, init, and platform controls without parsing tails |
| `sample.rs`                     | Uses extended `ExecRequest` with inherited cwd; keeps dev-only path and handoff behavior          |

`winter-cli/src/main.rs` injects the environment values below; dispatch code never calls `std::env` itself.

| Input            | Source                                     | Default when unset or empty                            |
| ---------------- | ------------------------------------------ | ------------------------------------------------------ |
| `cwd`            | `std::env::current_dir()`                   | Required; failure exits 126                            |
| `HOME`           | `var_os("HOME")`                            | Required for an unset XDG directory; failure exits 126 |
| `XDG_CONFIG_HOME` | `var_os("XDG_CONFIG_HOME")`, absolute only | `$HOME/.config`                                        |
| `XDG_CACHE_HOME` | `var_os("XDG_CACHE_HOME")`, absolute only   | `$HOME/.cache`                                         |
| Host target      | Build-time `TARGET`, exported by `build.rs` | Compiled in                                            |
| Legacy environment | `vars_os()` snapshot                    | Inherited `PATH`, optional `UV_PROJECT_ENVIRONMENT`     |
| Launch instant   | Wall-clock timestamp at launcher entry     | Decimal epoch seconds for legacy `WINTER_LAUNCH_TIME`  |

Relative XDG values are ignored, per the XDG base-directory specification. Legacy dispatch does not require XDG
directories, Rust version selections, or the cache. Platform help/version require none of these inputs. Tests isolate
the environment, synchronizer, cache, and complete ancestor filesystem (`engineering:fixture-only-dispatch`).

## Domain

```rust
use std::{ffi::OsString, path::PathBuf};

pub enum Role {
    Workspace, // group "workspace", executable "winter-workspace"
    Hub,       // group "hub",       executable "winter-hub"
    Runner,    // group "runner",    executable "winter-runner"
}

/// Exact release version, e.g. "1.4.0" or "2.0.0-rc.1". No ranges, aliases, or "latest".
/// Grammar: MAJOR.MINOR.PATCH[-PRERELEASE] with numeric parts lacking leading zeros and PRERELEASE in
/// [0-9A-Za-z.-]. Validation precedes any path join, so a version can never contain "/" or "..".
pub struct Version(String);

pub enum SelectionSource {
    Pin { file: PathBuf },       // nearest .winter/config.toml; Role::Workspace only
    Directory { file: PathBuf }, // .winterrc that supplied this role's winning value
    UserDefault { file: PathBuf },
}

pub struct Selection {
    pub role: Role,
    pub version: Version,
    pub source: SelectionSource, // carried into every dispatch diagnostic
}

pub struct DispatchRequest {
    pub role: Role,
    pub args: Vec<OsString>, // every token after the group, unchanged
}

pub struct DispatchEnv {
    pub cwd: PathBuf,
    pub config_home: PathBuf,
    pub cache_home: PathBuf,
    pub target: &'static str,
}

pub struct LegacyRequest {
    pub args: Vec<OsString>, // complete original tail, including command and leading Python options
    pub init_index: Option<usize>, // None = flag-gated legacy command; Some = temporary init token
}

pub struct LegacyEnv {
    pub cwd: PathBuf,
    pub inherited: Vec<(OsString, OsString)>,
    pub launch_time: OsString, // launcher-entry epoch seconds, not an inherited timestamp
}

pub enum Route {
    Modern(DispatchRequest),
    Legacy(LegacyRequest),
    // Foundation retains platform help, version, usage errors, and gated sample dispatch.
}

// Production extension of the foundation's ExecRequest; sample uses cwd=None and its existing environment policy.
pub struct ExecRequest {
    pub executable: PathBuf,
    pub args: Vec<OsString>,
    pub cwd: Option<PathBuf>, // None = inherited; legacy = workspace/tools/winter-cli
    pub env: Vec<(OsString, OsString)>, // explicit inherited snapshot plus route-specific changes
}
```

```text
selection order (decision:launcher-shim/version-scope)
  Workspace: nearest .winter/config.toml versions.workspace
               -> inherited .winterrc versions.workspace -> user default versions.workspace -> NoSelection
  Hub:       inherited .winterrc versions.hub             -> user default versions.hub       -> NoSelection
  Runner:    inherited .winterrc versions.runner          -> user default versions.runner    -> NoSelection

inherited .winterrc selection:
  enumerate cwd and every parent through filesystem root, inclusive
  read existing .winterrc files in root-to-cwd order
  merge recognized versions keys independently; each child value replaces that role's value and source file
  absent key/table -> leave inherited values unchanged; no file terminates the walk
  parse error / invalid version / lookup or read error other than not-found -> InvalidConfig
  validate every file, including values later overridden by a child
  HOME is not an ancestor-discovery input; ~/.winterrc participates only if it is an ancestor
  requested role absent after merge -> consult that role's user-selected default

user-selected default:
  read $XDG_CONFIG_HOME/winter/config.toml only when no applicable pin or directory selection exists
  versions.<role> present -> UserDefault; exact version, independent per role
  file/key absent -> NoSelection; include directory candidates and user file in searched paths
  lookup/read/parse error other than not-found / invalid recognized version -> InvalidConfig
  never infer a default from cache contents, PATH, or the platform executable's version
  selected release absent from cache -> NotCached, not a retry with a lower-precedence selection

Workspace pin:
  find nearest .winter/config.toml from cwd through root; stop at that file
  read only versions.workspace, not Workspace's full configuration model
  missing key -> inherited .winterrc selection, not an outer workspace's config
  parse error / invalid pin / lookup or read error other than not-found -> InvalidConfig
  valid pin -> select it without reading .winterrc
  Hub and Runner never discover or read .winter/config.toml
```

```text
# decision:singleton-launcher-succession
LEGACY_COMMANDS = {
  agents, capabilities, clean, dashboard, doctor, env, ext,
  graph, lint, provision, repo, service, space, ws
}

legacy discovery:
  find nearest .winter/config.toml from cwd through filesystem root; no outer-workspace fallback
  top-level enable-legacy-winter missing/false -> legacy disabled; true -> allow Python launch
  malformed TOML / non-boolean flag / lookup or read error other than not-found -> InvalidConfig
  require tools/winter-cli/ in that workspace; missing -> legacy failure, never run Rust instead
  no Rust version selection, cache manifest, XDG config, or user default is consulted

temporary init:
  require cwd/tools/winter-cli/; do not walk ancestors; do not read or require legacy flag
  create cwd/.winter/config.toml as an empty file only if absent; never truncate/replace existing content
  do not add enable-legacy-winter; subsequent ordinary legacy commands still require explicit opt-in
  replace the init token with two tokens, ws init; retain every other argument in place
```

## Interfaces

```text
winter workspace [ARG...]
winter hub [ARG...]
winter runner [ARG...]
winter [PYTHON-OPTION/ARG...] <LEGACY_COMMAND> [ARG...]
winter init [ARG...]                 # temporary legacy bootstrap; no flag required
winter --verbose|-v <GROUP> [ARG...]  # launcher diagnostics; legacy argv also reaches Python
```

```text
# Routing compares exact native OsStr tokens, not lossy strings or normalized clap values.
scan argv after argv[0] left-to-right for the first exact routing/control token:
  workspace | hub | runner   -> modern route; prefix may contain only --verbose/-v
  LEGACY_COMMANDS             -> flag-gated legacy route; preserve the WHOLE original argv tail
  init                       -> unconditional legacy bootstrap; preserve argv except init -> ws init
  --help | -h | --version     -> foundation controls; no configuration or cache reads
  sample                     -> existing foundation adapter, development builds only
  other platform commands    -> foundation owns their routing; never forward them to Python
no recognized token -> foundation usage error, exit 2; no implicit legacy fallback

stop classification at the routing token; never inspect or reclassify the selected application's tail
legacy prefixes are opaque: no allowlist of Python options or parsing of their values
the first routing/control token wins even in an opaque prefix; use --option=value for a value equal to a routing word
leading --winter / --winter=... is retired usage, exit 2; no checkout/source override is implemented
modern argv = unchanged tokens AFTER the group; root verbosity controls launcher stderr only
legacy argv = unchanged original tail, including root verbosity and Python-owned options
platform help lists modern groups, opt-in legacy routing, temporary init, and launcher verbosity
component help/version after a routing token belongs to that application, not the launcher
```

| Invocation | Route |
| ---------- | ----- |
| `winter workspace ws init` | Rust Workspace, argv `ws init`; legacy flag ignored |
| `winter --verbose hub --help` | Rust Hub, argv `--help`; launcher verbosity on stderr |
| `winter ws init` | Python, original argv `ws init`; legacy flag required |
| `winter --service-orchestrator ./alpha/provider service status alpha` | Python, entire original argv; no option parsing |
| `winter service --unknown-option runner` | Python; `runner` is not reclassified as a modern group |
| `winter init --json` | Temporary Python bootstrap, argv `ws init --json`; no flag/default required |
| `winter --help` | Platform help; no workspace, Python, default, or cache lookup |

```rust
pub enum DispatchError {
    NoSelection { role: Role, searched: Vec<PathBuf> },                // 127
    InvalidConfig { file: PathBuf, reason: String },                   // 126
    NotCached { selection: Selection, dir: PathBuf },                  // 127
    InvalidManifest { selection: Selection, file: PathBuf, reason: String }, // 126
    IncompatibleHandoff { selection: Selection, handoff: u32 },        // 126
    Replacement { selection: Selection, executable: PathBuf, source: std::io::Error }, // 126
}

pub fn resolve(role: Role, env: &DispatchEnv) -> Result<Selection, DispatchError>;
pub fn prepare(selection: &Selection, req: DispatchRequest, env: &DispatchEnv) -> Result<ExecRequest, DispatchError>;
pub fn run_dispatch(req: DispatchRequest, env: &DispatchEnv, process: &mut dyn Process)
    -> Result<std::convert::Infallible, DispatchError>;
pub fn dispatch_failure_reply(error: DispatchError) -> Reply;

pub enum LegacyError {
    NoWorkspace,                                      // 1
    Disabled { file: PathBuf },                        // 2; show flag and explicit modern entry point
    MissingCli { dir: PathBuf },                       // 1
    InvalidConfig { file: PathBuf, reason: String },    // 126
    Setup { path: PathBuf, source: std::io::Error },    // 126; includes bootstrap/venv/stamp effects
    Sync { executable: PathBuf, status: std::process::ExitStatus }, // propagate nonzero status/signal
    Replacement { executable: PathBuf, source: std::io::Error },    // ENOENT=127; other errors=126
}

pub trait LegacySetup {
    // Implements the legacy/init flow below through injected filesystem and synchronizer adapters.
    fn prepare(&mut self, req: LegacyRequest, env: &LegacyEnv) -> Result<ExecRequest, LegacyError>;
}

// Main injects the inherited environment and launch instant; adapters own filesystem and uv effects.
pub fn run_legacy(req: LegacyRequest, env: &LegacyEnv, setup: &mut dyn LegacySetup, process: &mut dyn Process)
    -> Result<std::convert::Infallible, LegacyError>;
```

| Outcome                    | stdout      | stderr                                                                       | Exit                         |
| -------------------------- | ----------- | ---------------------------------------------------------------------------- | ---------------------------- |
| Replacement succeeds       | Component's | Component's                                                                  | Component's status or signal |
| `NoSelection`, `NotCached` | Empty       | `winter: cannot run <role>: …`, the files searched or the expected directory | 127                          |
| `InvalidConfig`            | Empty       | `winter: cannot run <role>: …`, the invalid file and reason                 | 126                          |
| Any other `DispatchError`  | Empty       | `winter: cannot run <role> <version> (from <source file>): …`                | 126                          |

Every dispatch diagnostic starts with `winter: cannot run`; legacy failures use `winter: cannot run legacy workspace`
with the relevant workspace/path and cause, not a fabricated Rust version. Modern dispatch never retries another
version, searches PATH, or writes files (`product:explain-dispatch-failure`). Legacy setup may locate `uv` on inherited
PATH, synchronize its venv, write its sync stamp, and perform temporary init, but never searches PATH for a product.

| Handoff field  | Contract                                                                                            |
| -------------- | --------------------------------------------------------------------------------------------------- |
| Executable     | `<cache>/winter/versions/<role>/<version>/<target>/<manifest.executable>`, direct `execve`, no PATH |
| `argv[0]`      | The selected executable path                                                                        |
| Arguments      | Every token after the group, unchanged `OsString` values                                            |
| cwd, fds 0/1/2 | Inherited unchanged                                                                                 |
| Environment    | Inherited snapshot plus `WINTER_INVOKED_AS=winter <group>`, replacing any inherited value           |
| Process        | Same PID via `execve`; the component owns exit status and signals (`product:preserve-invocation`)   |

```text
# component side of the handoff, handoff = 1 (product:invocation-aware-help)
WINTER_INVOKED_AS set and nonempty -> use it as the usage prefix in help, nested help, and usage errors
WINTER_INVOKED_AS unset or empty   -> use the component's own executable name
read once at startup, then removed from the component's environment so child processes never inherit it
never affects component identity, --version output, or behavior other than displayed prefixes
```

The shim supports handoff version `1` only. A later handoff version adds to this table and keeps accepting `1`.

| Legacy handoff field | Contract |
| -------------------- | -------- |
| Executable | Absolute `<resolved venv>/bin/winter`; direct `execve`, never the old Bash wrapper or `winter` from PATH |
| Arguments | Original legacy argv; only temporary init rewrites its routing token to `ws init` |
| cwd | `<workspace>/tools/winter-cli`, retaining the Python GitPython layout workaround |
| Invocation directory | `WINTER_INVOCATION_CWD` replaced with original caller cwd before any chdir |
| Python environment | Remove `PYTHONHOME`; set `VIRTUAL_ENV`; prepend venv `bin` to inherited `PATH` |
| Launch timing | Replace `WINTER_LAUNCH_TIME` with launcher-entry epoch seconds for Python's existing tracing reader |
| Invocation metadata | Remove inherited `WINTER_INVOKED_AS`; legacy Python keeps its existing `winter` usage prefix |
| Streams and process | Inherit fds 0/1/2; same PID after exec; Python owns exit status and signals |
| Synchronizer | `uv sync --project <absolute cli dir>`, no shell, cwd = cli dir, all setup output redirected to stderr |
| Setup failure/cancellation | Do not exec Python; preserve failure status or interruption and terminate/reap setup children |

## Flows

```text
args_os + launcher-entry instant -> classify routing token
  platform control -> foundation reply/adapter; help/version need no cwd or config
  legacy/init      -> capture cwd and environment snapshot -> run_legacy (flow below)
  modern group     -> capture cwd, environment snapshot, and XDG directories
       -> DispatchRequest { role, args = unchanged tokens after group }
      -> resolve(role)
            Workspace only: walk cwd, parent, ... "/" for nearest .winter/config.toml
                         lookup/read/parse error or bad pin -> InvalidConfig
                         found + versions.workspace         -> Pin; skip .winterrc
                         found without key / no file         -> resolve inherited .winterrc
            inherited .winterrc:
                         enumerate ancestor paths through "/" inclusive
                         read and validate root to cwd; missing files are skipped
                         lookup/read/parse error or bad version -> InvalidConfig
                         each versions key replaces only its role's value and source
                         requested role present                -> Directory { file: winning source }
                         requested role absent                 -> user config versions.<role>
                             file/key missing                  -> NoSelection (all searched paths listed)
                             malformed/unreadable/bad version  -> InvalidConfig
                             key present                       -> UserDefault
      -> prepare
           dir = cache/winter/versions/<role>/<version>/<target>/
           dir/component.toml missing                  -> NotCached
           parse error, role/version/target mismatch   -> InvalidManifest
           handoff not in {1}                          -> IncompatibleHandoff
           executable relative path escapes dir        -> InvalidManifest
            -> ExecRequest { executable, args, cwd: None, env: inherited + WINTER_INVOKED_AS }
      -> Process.replace
           error -> Replacement -> dispatch_failure_reply -> exit 126
           ok    -> component owns the process
```

No preflight checks the executable's existence or mode; `execve` reports `ENOENT` and `EACCES` as `Replacement`, which
avoids a check-then-exec race. `.winterrc` discovery never stops at the first file; a lookup or read error other than
not-found, such as an unreadable ancestor, is `InvalidConfig`, not permission to skip that ancestor.

```text
legacy/init:
  init -> cwd/tools/winter-cli exists?
            no  -> MissingCli; no ancestor discovery or config mutation
            yes -> create .winter/ and empty config only if absent, preserving any existing file
                   root = cwd; rewrite init token to ws init
  other legacy command -> discover nearest config; require enable-legacy-winter = true
                          root = config parent directory's parent; require root/tools/winter-cli
  cli = root/tools/winter-cli
  setup_env = inherited snapshot with original WINTER_INVOCATION_CWD and launcher-entry WINTER_LAUNCH_TIME,
              removing inherited WINTER_INVOKED_AS
  venv = inherited UV_PROJECT_ENVIRONMENT when nonempty, otherwise cli/.venv
         relative venv paths resolve against cli, not original cwd
  stamp = venv/.winter-sync-stamp
  stale = entrypoint not executable OR stamp missing OR any existing
          cli/{uv.lock,pyproject.toml,.python-version} has mtime newer than stamp
  stale?
    no  -> do not locate or run uv
    yes -> resolve uv on inherited PATH, then run uv sync --project cli in setup_env,
           with setup stdout/stderr on stderr
           fail/cancel -> no stamp refresh, no Python launch; report failed setup
           success     -> touch stamp; a failed stamp write also prevents Python launch
  exec_env = setup_env without PYTHONHOME, VIRTUAL_ENV = venv, PATH = venv/bin prepended to inherited PATH
  Process.replace(venv/bin/winter, original/re-written args, cwd = cli, env = exec_env)
    failure -> LegacyError::Replacement; never retry another implementation
    success -> Python owns process, output, exit status and signals

# Modern paths never enter this flow, even when enable-legacy-winter is true.
# Init and uv sync are the only documented exceptions to modern dispatch's no-write/no-install contract.
# No init failure rolls back by deleting an existing config or other operator-owned content.
```

## Data

```toml
# <workspace root>/.winter/config.toml — top-level compatibility flag, independent of Rust pin
enable-legacy-winter = true

[versions]
workspace = "1.4.0"
# modern workspace reads only versions.workspace; legacy reads only enable-legacy-winter
# hub/runner read neither field and never discover this file
```

```toml
# /srv/.winterrc — platform-owned selections; no Winter workspace required
[versions]
workspace = "1.4.0"
hub = "2.0.0"
runner = "1.9.2"
```

```toml
# /srv/deployment/.winterrc — overrides Runner only
[versions]
runner = "1.9.3"
# cwd /srv/deployment or below inherits hub = "2.0.0" and workspace = "1.4.0" from /srv/.winterrc
# a Workspace pin in the nearest .winter/config.toml takes precedence over inherited versions.workspace
```

```toml
# $XDG_CONFIG_HOME/winter/config.toml — platform-owned, user-selected defaults, not Workspace configuration
[versions]
workspace = "1.3.0"
hub = "1.8.0"
runner = "1.9.0"
# consulted for a role only when it has no applicable pin or inherited directory selection
```

```toml
# $XDG_CACHE_HOME/winter/versions/runner/1.9.2/x86_64-unknown-linux-gnu/component.toml
# decision:launcher-shim/component-discovery
role = "runner"
version = "1.9.2"
target = "x86_64-unknown-linux-gnu"
handoff = 1
executable = "bin/winter-runner" # relative; normalized result must stay inside this directory
```

| File                  | Unknown table or key | `versions.hub` / `versions.runner` | Version string fails grammar |
| --------------------- | -------------------- | --------------------------------- | ---------------------------- |
| `.winterrc`           | Ignored              | Merged independently              | `InvalidConfig`              |
| `.winter/config.toml` | Ignored by shim      | Ignored by shim                   | `InvalidConfig` for pin only |
| User `winter/config.toml` | Ignored          | Used as role defaults             | `InvalidConfig`              |
| `component.toml`      | Ignored              | n/a                               | `InvalidManifest`            |

Unknown keys are ignored so later fields can be added without breaking older shims. A `.winterrc` merges only recognized
`versions` keys, not unrelated tables or arbitrary Workspace configuration. Modern dispatch never writes these files;
temporary init may create an absent workspace config but never edits an existing one. Legacy setup writes only its
dependency environment and sync stamp in addition to that bootstrap exception.
