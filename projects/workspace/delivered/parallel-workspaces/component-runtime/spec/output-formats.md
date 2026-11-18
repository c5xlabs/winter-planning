# Output formats

What the binary writes and where: stream assignment, human-output conventions, the three JSON shapes, and which command
uses which. Reference locations are paths under `tools/winter-cli/src/winter_cli/` of the `winter-workspace` repository
at `b57b020`. Errors and exit codes are [results-and-exit-codes.md](./results-and-exit-codes.md); the command tree is
[command-surface.md](./command-surface.md); the owners that write to each stream are [runtime.md](./runtime.md).

Under `decision:functional-parity` the contract is the stream, the JSON documents as data, and the outputs another
program reads. Wording, layout, glyphs, and styling of human output are the binary's own; where the reference tool's
form is shown it is an example.

## Interfaces

### Streams

| Content                                                                | Stream |
| ---------------------------------------------------------------------- | ------ |
| A command's result, human or JSON                                      | stdout |
| `--help`, `--version`                                                  | stdout |
| Confirmation prompts and their previews                                | stdout |
| The plain-text notice that the given patterns matched nothing          | stdout |
| Help of a bare group; every error diagnostic; the aborted notice       | stderr |
| Warnings, log records, reporter failure lines, transient-retry notices | stderr |
| The notice that a service orchestrator override is in effect           | stderr |

There is no pager, spinner, progress bar, or carriage-return redraw outside `dashboard`. Long operations write one line
per reporter event as it happens.

```text
# engineering:machine-output-boundary — runtime.md gives each owner's signature
stdout   written only through crates/core/src/output.rs and crates/core/src/confirm.rs
stderr   written by the library only through crates/core/src/diagnostics.rs, and by the process boundary for the
         final diagnostic, the backtrace, and the aborted notice
a failed write to stdout ends the invocation (results-and-exit-codes.md, A stdout that cannot be written);
a failed write to stderr changes nothing
```

### Human output

```text
# decision:parallel-workspaces/component-runtime/color-policy
colour:   ANSI styling is written only when the target stream is a terminal; decided per stream
          NO_COLOR and FORCE_COLOR are not read; there is no --color option
          help, the version line, and every diagnostic are never styled
width:    no output depends on the terminal's width; COLUMNS is not read
encoding: UTF-8, unconditional; the locale is not read. The reference tool takes its stream encoding from its
          language runtime, so this rule has no reference source
```

Tables, glyphs, styles, headers, and column layout are not contract. Each command's human result carries the same
information as its `--json` document where it has one; its layout belongs to its epic's spec.

Results another program reads, whose form is contract:

| Command          | stdout                                                                                  |
| ---------------- | --------------------------------------------------------------------------------------- |
| `ws fingerprint` | the digest alone, one line                                                              |
| `ws index`       | the integer first on the line; a suggested, not yet allocated, index is marked after it |
| `space`          | one absolute path, one line                                                             |
| `env`            | `export KEY=<shell-quoted value>` lines a POSIX shell can source                        |
| `ws diff`        | unified diffs `git apply` accepts with `--no-headers`; otherwise a header line per repo |
| `service logs`   | relayed log lines                                                                       |

### Help text

```text
# decision:parallel-workspaces/component-runtime/help-rendering
# decision:parallel-workspaces/component-runtime/program-name
content:   every visible command and option of command-surface.md that the build holds, each with its value
           placeholder and a description; hidden ones omitted
name:      the program is named "winter-workspace", whatever name the binary was invoked under
wording, order, wrapping, alignment: the argument parser's own; not contract
```

### JSON shapes

```text
# product:automation-boundary, decision:parallel-workspaces/component-runtime/json-encoding
There is no common envelope and no schema-version field, except `ws status`, whose document carries
`schema_version: 1` and is the only one with a published schema.
```

| Shape | Form                                                                           | Reference encoder                                                                                                                                                                                                                                                                        |
| ----- | ------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **A** | One document; it may span lines.                                               | `modules/workspace/handlers/json_render.py:31-32`, `repo_handler.py:80,168,194`, `modules/service/service_reporter.py:164`                                                                                                                                                               |
| **B** | One document per line.                                                         | `modules/capability/capability_reporter.py:131`, `modules/agents/matrix_reporter.py:234`, `modules/graph/graph_reporter.py:44`, `modules/ext/verify_reporter.py:81`, `modules/service/command.py:398,406,431`                                                                            |
| **C** | NDJSON: one object per line, each with a `type` key, written as events happen. | `modules/workspace/init_reporter.py:127`, `fetch_reporter.py:77`, `pull_reporter.py:108`, `merge_reporter.py:91`, `handlers/workspace_handler.py:1566-1637`, `modules/doctor/doctor_reporter.py:113`, `modules/lint/lint_reporter.py:207`, `modules/provision/provision_reporter.py:267` |

What is contract, common to all three shapes:

```text
encoding        any valid JSON encoding of the document, in UTF-8; whitespace, indentation, the order of keys in
                an object, and the choice between a literal character and an escape are not contract
framing         B and C: exactly one document per line, so no line break inside a document
                C: each event is written and flushed when it happens, not at the end of the run
keys            snake_case; the key set of each document and event is fixed by its owning spec
values          the same JSON type and value as the reference tool's document
numbers         an integer stays an integer; a float stays a float
nulls           a null is written explicitly ("error": null); a key is absent only where its document says so
arrays          element order is contract
paths           absolute, as strings
enums           their string value
message text    a value that is a human message (an `error`, `notes`, `message`, or output-line field) is a
                string in the same place; its wording is as free as a message on stderr
```

```text
writers                       # one encoder, one owner: crates/core/src/output.rs
shape A, one document         output::document                      command-registration.md, Writing a result
shape B                       one Results::document call per line   runtime.md, Writing more than one piece
shape C                       one Results::event call per event; each is flushed
events, then a document       Results::event calls, then Results::document
a human result of many lines  Results::line calls
```

### Shape per command

| Shape | Commands                                                                                                                                                                                                                                                      |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A     | `ws list` (array), `ws status` (object), `ws worktrees` (array), `ws connect`, `ws disconnect`, `ws checkout`, `ws push`, `ws index`, `ws prune`, `ws diff`, `ws restack`, `ws fingerprint`, `repo list` (array), `repo add`, `repo remove`, `service status` |
| B     | `capabilities` (array), `agents` (object), `graph` (object), `ext verify` (one object per operand, so N lines for N operands), `service catalog`, `service ext-services`                                                                                      |
| C     | `ws init`, `ws destroy`, `ws fetch`, `ws pull`, `ws update`, `ws merge`, `ws reset`, `ws clean`, `doctor`, `lint`, `provision`, `clean`                                                                                                                       |

NDJSON event vocabularies. The field set of each event and document belongs to the spec of the epic that owns the
command.

| Command                 | `type` values, in the order a run can emit them                                                                                                                                                     |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ws init`, `ws destroy` | `target_started`, `repo_action`, `repo_error`, `cmd_started`, `cmd_output_line`, `cmd_completed`, `target_completed`                                                                                |
| `ws fetch`              | `fetch_started`, `repo_fetched`, `fetch_completed`                                                                                                                                                  |
| `ws pull`, `ws update`  | `pull_started`, `repo_synced`, `env_skipped`, `pull_completed`                                                                                                                                      |
| `ws merge`              | `merge_started`, `repo_merged`, `merge_completed`                                                                                                                                                   |
| `ws reset`              | `reset_started`, `repo_reset`, `reset_completed`                                                                                                                                                    |
| `ws clean`              | `clean_started`, `repo_clean`, `clean_completed`                                                                                                                                                    |
| `doctor`                | `started`, `probe_result`, `finished`                                                                                                                                                               |
| `lint`                  | `started`, `finding`, `ignored`, `finished`                                                                                                                                                         |
| `provision`, `clean`    | `started`, `subtarget_started`, `no_handlers`, `execution_started`, `execution_output_line`, `execution_completed`, `execution_error`, `handler_result`, `handler_warn`, `plan_handler`, `finished` |

`cmd_output_line` and `execution_output_line` carry the child's stdout and stderr lines merged in arrival order
([invocation contract](../../../../work/agent-ready-operation/executable-extensions/spec/invocation-contract.md)).

Documents held here until the spec of the epic that owns each command exists; the owner is named on the right, and the
placement is temporary (Open questions):

```text
repo add --json       {"added": true, "kind": "project"|"standalone", "url": <string>}                         # epic:parallel-workspaces/repository-catalog
repo remove --json    {"removed": true, "kind": "project"|"standalone", "name": <string>}                      # epic:parallel-workspaces/repository-catalog
service catalog       {"services": [...]}; when no provider resolves: {"services": [], "error": <message text>}, exit 0
                                                                                                               # epic:running-environments/service-orchestration
service ext-services  {"services": [{"name": <string>, "scope": <string>, "source": <string>}]}                # epic:running-environments/service-orchestration
ws prune --json       --dry-run: {"dry_run": true, "orphans": [...]}     else: {"dry_run": false, "orphans": [...], "excludes_updated": <bool>}
                      orphan: {"kind", "path", "safe_to_remove", "notes", "action"} plus "error" when action is "error"
                      action: "would_remove" | "skipped" under --dry-run; "removed" | "skipped" | "error" otherwise
                                                                                                               # epic:parallel-workspaces/workspace-bootstrap
```

### JSON-mode departures

Places where stdout under `--json` is not, or not only, the command's document
(`decision:parallel-workspaces/component-runtime/json-mode-purity`).

| Case                                                        | stdout under `--json`                                                                                      | Reference                                                                                                           |
| ----------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `provision`, `clean`, `ws destroy`, `lint` matching nothing | one plain-text line naming the patterns; no JSON; exit 0                                                   | `modules/provision/handler.py:78`, `modules/workspace/handlers/destroy_handler.py:56`, `modules/lint/handler.py:52` |
| `ws status --fetch --json`                                  | shape C fetch events, then the shape A document                                                            | `modules/workspace/handlers/workspace_handler.py:276-306`                                                           |
| `ws prune --json` without `--dry-run`                       | shape C init events may precede the shape A document                                                       | `modules/workspace/handlers/workspace_handler.py:1036-1038`                                                         |
| `ws destroy`, `ws reset --hard` needing confirmation        | the plain-text preview and prompt, unless `--force` or `--dry-run`                                         | `destroy_handler.py:59-61`, `workspace_handler.py:497-502`                                                          |
| `ws clean --json` without `--force` or `--dry-run`          | nothing; a command error                                                                                   | `workspace_handler.py:585-591`                                                                                      |
| `ws fetch --json` in a workspace with no repos              | nothing at all                                                                                             | observed                                                                                                            |
| `ws worktrees --json` without `--status`                    | `ahead`, `behind`, `dirty` keys are absent, not null                                                       | `modules/workspace/handlers/workspace_handler.py:1527-1546`                                                         |
| Enum spelling across documents                              | mixed, and kept: `refused-unknown-branch`, `fork-point` beside `no_upstream`, `up_to_date`, `would_remove` | observed                                                                                                            |
| Any error                                                   | nothing; see [results-and-exit-codes.md](./results-and-exit-codes.md)                                      | `cli.py:430-450`                                                                                                    |

### Diagnostics

All on stderr; their wording and prefixes are not contract.

| Kind                                             | When                                                           | Reference                                                                                                                 |
| ------------------------------------------------ | -------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| Drift warning (configuration against filesystem) | before a command that acts on worktrees, when the two disagree | `modules/workspace/drift.py:83-88`                                                                                        |
| Log record, WARNING or above, no logging switch  | written as a plain line carrying the message alone             | `decision:parallel-workspaces/component-runtime/unswitched-log-records`; 61 warning sites                                 |
| Log record, logging switch set                   | written with its level and origin, at or above the level set   | `cli.py:264-293`                                                                                                          |
| Reporter failure lines                           | one per failed repository, command, or stage, as it happens    | `modules/workspace/init_reporter.py:42,62,100,111`, `fetch_reporter.py:45`, `modules/provision/provision_reporter.py:118` |
| Transient git retry                              | one per retried attempt, naming the attempt and the delay      | `modules/workspace/internal/git_ops_service.py`                                                                           |

Warnings are never folded into a JSON document or event. A failed git call is not written as a bare record when no
logging switch is set, under `decision:parallel-workspaces/component-runtime/git-failure-rendering`. No record is
written by an invocation that ends at the root ([results-and-exit-codes.md](./results-and-exit-codes.md), Flows).

| Switch                                          | Effect                                                                                                                                                                                             |
| ----------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-v`, `--verbose`                               | Records at DEBUG and above on stderr. Wins over the variable.                                                                                                                                      |
| `WINTER_LOG_LEVEL=<NAME>`                       | Same at the named level, any case: `DEBUG`, `INFO`, `WARNING`, `WARN`, `ERROR`, `CRITICAL`, `FATAL`, `NOTSET` ([runtime.md](./runtime.md), Logging). An unknown or empty name is ignored silently. |
| neither                                         | Records at WARNING and above are written; lower levels are dropped. Tracing-exporter records are dropped.                                                                                          |
| `RUST_BACKTRACE`, `RUST_LIB_BACKTRACE`          | Set to anything but `0`: a backtrace follows an internal-failure diagnostic. Read by nothing else.                                                                                                 |
| `WINTER_OTEL_EXPORTER_OTLP_ENDPOINT`            | Non-blank turns tracing on: OTLP over HTTP with protobuf to `<endpoint>/v1/traces`, one export at exit, wait capped at 100 ms.                                                                     |
| `WINTER_OTEL_EXPORTER_OTLP_HEADERS`             | Export request headers, `k1=v1,k2=v2`.                                                                                                                                                             |
| `OTEL_SDK_DISABLED=true`                        | Turns tracing off; only the value `true`, any case, surrounding blanks ignored.                                                                                                                    |
| `TRACEPARENT`                                   | Parent of the command span; its sampled flag decides recording.                                                                                                                                    |
| `WINTER_LAUNCH_TIME`                            | Start instant of the command span when set and valid; otherwise the span starts at dispatch ([command-surface.md](./command-surface.md), Process inputs).                                          |
| `OTEL_SERVICE_NAME`, `OTEL_RESOURCE_ATTRIBUTES` | Span resource; `service.name` defaults to `winter`.                                                                                                                                                |
| `OTEL_EXPORTER_OTLP_*`                          | Not read by the exporter; passed unchanged to child processes.                                                                                                                                     |

The command span is named `winter <command path>` (`winter ws init`), whatever the binary is called (`cli.py:138-140`;
`decision:parallel-workspaces/component-runtime/program-name`). The path is the command names the tokens resolved,
stopping at the first option, positional, or unknown name. Tracing never writes to stdout and never changes an exit code
(`engineering:observability-isolation`).

## Open questions

- Which JSON string fields are message text is stated above by field name only. Engineering lists the fields per
  document as each owning command's spec is written.
- `ws status --json` documents were extended by in-process plugin decorators in the reference tool
  (`modules/workspace/handlers/workspace_handler.py:284-296`). With plugin loading dropped, the document carries only
  core fields; engineering confirms against `schemas/ws-status-v1.json` when the status spec is written.
- The `repo add`, `repo remove`, `service catalog`, `service ext-services`, and `ws prune` documents under Shape per
  command are placed here temporarily. Engineering moves each into the spec of the epic named beside it when that epic
  is refined, and this leaf then links it.
