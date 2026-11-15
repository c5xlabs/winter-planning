# Sample handoff and executable

## Domain

```rust
use std::{convert::Infallible, ffi::OsString, io, path::PathBuf};

// All sample/process exports below are cfg(feature = "dev-sample") in winter-platform.
pub const SAMPLE_PATH_ENV: &str = "WINTER_DEV_SAMPLE_PATH";

pub struct SampleRequest {
    pub args: Vec<OsString>,
}

pub struct ExecRequest {
    pub executable: PathBuf,
    pub args: Vec<OsString>,
}

pub enum LaunchError {
    MissingPath,
    RelativePath(PathBuf),
    Replacement { executable: PathBuf, source: io::Error },
}

pub trait Process {
    fn replace(&mut self, request: ExecRequest) -> io::Result<Infallible>;
}

pub struct UnixProcess;

pub fn run_sample(
    request: SampleRequest,
    executable: Option<OsString>, // value injected by main, not read by command behavior
    process: &mut dyn Process,
) -> Result<Infallible, LaunchError>;
```

## Interfaces

| Handoff field                  | Contract                                                                                                                     |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------- |
| `WINTER_DEV_SAMPLE_PATH`       | Absolute native path; unset/empty is `MissingPath`, relative is `RelativePath`                                               |
| Executable selection           | Direct `execve`; no PATH search, shell fallback, canonicalization, downloads, or existence/permission preflight              |
| Sample `argv[0]`               | Selected executable path; not the original launcher's name                                                                   |
| Remaining arguments            | Unchanged `OsString` values; no UTF-8 conversion, shell, escaping, or delimiter removal                                      |
| cwd and file descriptors 0/1/2 | Inherited without changing directory, wrapping streams, or buffering sample output                                           |
| Environment                    | Explicit inherited environment snapshot excluding `WINTER_DEV_SAMPLE_PATH`; no mutation of the launcher's global environment |
| Successful replacement         | Same PID, no return to launcher; sample owns exit code, diagnostics, and signals                                             |
| Failed replacement             | `LaunchError::Replacement`; launcher writes stderr diagnostic including the OS error, exits `1`                              |

```text
UnixProcess::replace:
  executable = CString from absolute native path bytes
  argv storage = CString vector [selected executable path, unchanged argument bytes...]
  env storage = CString vector of native key=value bytes from std::env::vars_os(),
                excluding the exact WINTER_DEV_SAMPLE_PATH key
  argv/envp = pointer vectors into their respective storage, each terminated by NULL
  reject embedded NUL as io::ErrorKind::InvalidInput; keep all storage alive through the call
  unsafe libc::execve(executable.as_ptr(), argv.as_ptr(), envp.as_ptr())
    success -> never returns
    return  -> Err(io::Error::last_os_error()), captured immediately
```

`execvp` and `CommandExt::exec` are not used for this handoff: invalid executable format must fail rather than trigger a
userspace shell fallback. An executable text fixture without a shebang produces a launcher diagnostic and exit `1`; its
marker-writing body must not run. Kernel-recognized interpreter scripts retain normal `execve` semantics.

The component double records `ExecRequest` and returns a controlled `io::Error`; it does not fake successful replacement
with a normal return. Runtime invocation metadata for future product help is not added to the sample handoff.

```text
winter-dev-sample
winter-dev-sample --help
winter-dev-sample inspect [ARG...]
winter-dev-sample copy-stdin
winter-dev-sample emit
winter-dev-sample exit CODE
winter-dev-sample terminate
winter-dev-sample wait
```

| Sample mode                           | stdout                            | stderr                  | Termination                                                    |
| ------------------------------------- | --------------------------------- | ----------------------- | -------------------------------------------------------------- |
| No arguments / `inspect [ARG...]`     | One JSON inspection record and LF | Empty                   | `0`                                                            |
| `--help`                              | Sample usage, not launcher usage  | Empty                   | `0`                                                            |
| `copy-stdin`                          | Exact stdin bytes through EOF     | Empty                   | `0`, or `1` on I/O failure                                     |
| `emit`                                | `sample stdout\n`                 | `sample stderr\n`       | `0`                                                            |
| `exit CODE`                           | `sample exit <CODE>\n`, flushed   | Empty                   | Requested integer `0..255`                                     |
| `terminate`                           | `ready\n`, flushed                | Empty                   | Self-termination with default, unblocked `SIGTERM`             |
| `wait`                                | `ready\n`, flushed                | Empty                   | Blocks for harness-sent `SIGINT`/`SIGTERM` or deadline cleanup |
| Unknown mode / invalid mode arguments | Empty                             | Sample usage diagnostic | `2`                                                            |

`inspect` does not parse its remaining operands; even `--` is reported as an argument. Other modes reject unexpected
operands. The sample has no arbitrary-command mode, configuration loading, or complete environment dump.

An initial sample `--` is invalid usage: `winter sample -- --help` exits `2`, while `winter sample inspect -- --help`
reports both tokens literally. `wait` resets and unblocks `SIGINT` and `SIGTERM` before flushing readiness; ignored
dispositions or blocked masks inherited from its caller do not disable signal tests.

```json
{
  "pid": 123,
  "argv0_hex": "2f746d702f77696e7465722d6465762d73616d706c65",
  "args_hex": ["696e7370656374", "ff"],
  "cwd_hex": "2f746d70",
  "env_hex": {
    "WINTER_SAMPLE_PROBE": "74657374",
    "WINTER_DEV_SAMPLE_PATH": null
  }
}
```

Unix paths, arguments, and selected environment values use lowercase hex of their original bytes, not lossy UTF-8.
`args_hex` contains every argument after sample `argv[0]`, including `inspect`; absent environment values are `null`.
Only the two named probe variables are inspected. JSON key order is not a contract.

## Flows

```text
SampleRequest + injected path
  -> validate nonempty absolute path, without filesystem preflight
  -> ExecRequest
  -> injected Process.replace
       failure -> typed error -> launcher reply(stderr, exit 1)
       success -> launcher image replaced -> sample mode owns remaining behavior

service harness -> spawn development winter -> capture spawned PID
  -> winter exec sample -> inspect.pid == spawned PID
  -> inspect/copy/emit/exit/terminate/wait -> observe raw streams and OS termination
```
