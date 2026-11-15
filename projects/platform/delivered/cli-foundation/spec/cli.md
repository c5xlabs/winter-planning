# CLI parsing and replies

## Interfaces

```text
production:
  winter
  winter -h | --help
  winter -V | --version

development, with dev-sample compiled in:
  winter sample [ARG...]
```

| Invocation                                                    | stdout                                  | stderr                                             | Exit                      |
| ------------------------------------------------------------- | --------------------------------------- | -------------------------------------------------- | ------------------------- |
| No arguments, `-h`, `--help`                                  | Identical help bytes, ending in LF      | Empty                                              | `0`                       |
| `-V`, `--version`                                             | `winter <winter-cli package version>\n` | Empty                                              | `0`                       |
| Unknown command/option or unexpected operand rejected by clap | Empty                                   | Usage error and `--help` pointer                   | `2`                       |
| Production `sample`, regardless of environment or PATH        | Empty                                   | Unsupported-command error and help pointer         | `2`                       |
| Development `sample [ARG...]`                                 | Owned by sample after replacement       | Owned by sample after replacement                  | Sample's status or signal |
| Development sample-path or replacement failure                | Empty                                   | Launcher diagnostic identifying the failed handoff | `1`                       |
| Launcher reply cannot be written                              | No additional stdout                    | Best-effort I/O diagnostic                         | `1`                       |

```text
# product:discover-help, product:identify-version, product:unsupported-invocation
parser = clap 4, builder API
display name = bin name = "winter", independent of argv[0]
color = never; help width = 100 columns; no terminal-width lookup
help/version = clap's built-in actions and standard argument handling
automatic help subcommand = disabled
help aliases = -h / --help; version aliases = -V / --version
no arguments, or successful root parse with no action = Help
development help adds "winter sample [ARG...]"; production help contains no sample registration
```

Repeated or combined flags, control-option precedence, and root `--` follow clap's standard behavior without custom
validation. Help/error wording belongs to the parser module; tests compare bare/explicit help exactly but match semantic
error fragments rather than freezing all clap punctuation.

```rust
use std::ffi::OsString;

pub struct CliIdentity {
    pub version: &'static str, // injected by winter-cli/main.rs
}

pub enum Control {
    Help,
    Version,
}

pub enum Request {
    Control(Control),
    #[cfg(feature = "dev-sample")]
    Sample(SampleRequest),
}

pub struct Reply {
    pub stdout: Vec<u8>,
    pub stderr: Vec<u8>,
    pub exit_code: u8,
}

pub fn parse(argv: impl IntoIterator<Item = OsString>, identity: &CliIdentity) -> Result<Request, UsageError>;
pub fn control_reply(request: Control, identity: CliIdentity) -> Reply;
pub fn usage_reply(error: UsageError) -> Reply;
#[cfg(feature = "dev-sample")]
pub fn launch_failure_reply(error: LaunchError) -> Reply;
```

`argv` includes the executable's `argv[0]`; parse discards it without decoding it. `UsageError` carries parser
diagnostics, not a process exit side effect; `main` writes returned bytes without extra newlines or prefixes and returns
the reply's exit code.

After the sample adapter discards `argv[0]`, clap uses `no_binary_name(true)` so it does not discard a second token. The
shared command builder receives the executable identity for clap's version action and reply rendering; `DisplayHelp` and
`DisplayVersion` become successful control requests, not usage errors or parser-triggered process exits.

## Flows

```text
args_os()
  -> drop argv[0]
  -> cfg(dev-sample) AND first token == OsStr("sample")?
       yes: Request::Sample { args = every remaining OsString, unchanged }
       no: clap parses root inspection flags; unsupported input -> UsageError
  -> Control: render reply without reading environment or probing the filesystem
  -> Sample: read WINTER_DEV_SAMPLE_PATH with var_os, then call sample handoff
```

| Development invocation                        | Sample receives after its own `argv[0]`      |
| --------------------------------------------- | -------------------------------------------- |
| `winter sample`                               | Empty argument vector                        |
| `winter sample --help`                        | `--help`                                     |
| `winter sample --version`                     | `--version`                                  |
| `winter sample -- --help`                     | `--`, `--help`                               |
| `winter sample inspect --unknown <non-UTF-8>` | `inspect`, `--unknown`, original byte string |

The feature-gated first-token adapter and its help annotation live at one registration site in `cli.rs`; the raw tail
does not pass through clap normalization. `winter -- sample` is not a dispatch spelling, and the adapter does not exist
in production builds.
