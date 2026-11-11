# CLI context specification

For `epic:cli-context`.

## System

```text
Blizzard repository                         Distribution build
───────────────────                         ──────────────────
cli/context/topics/**/*.md ────────────────► Markdown/catalog compiler
  UTF-8 body + YAML frontmatter                │ validate sources and links
                                              │ generate immutable topic records
                                              ▼
                                            cli/context/_builtin.py
                                              │ included in wheel and editable builds
                                              ▼
Installed Blizzard CLI
──────────────────────
blizzard context [TOPIC]
  └─ packaged catalog → exact topic / directory-index alias → Markdown → stdout

No runtime edges to: hub, runner, artifact scopes, graph mints, prompt configuration,
                    worker identity, user config, source checkout, or network.
```

| Component        | Contract                                                                                                  |
| ---------------- | --------------------------------------------------------------------------------------------------------- |
| CLI registration | Add a lazy `context` target to `blizzard.cli.main`; leave existing targets unchanged.                     |
| Source compiler  | Build-time only; produces the packaged catalog from the Markdown source format in [Data](#data).          |
| Catalog lookup   | Exact topic keys and compiled directory-index aliases; no filesystem lookup, decoding, or online refresh. |
| Output           | Construct and validate the complete response before writing; no pager or ANSI renderer.                   |
| Version          | Read the installed CLI package version; do not infer a remote hub version.                                |

## Domain

```python
from collections.abc import Mapping
from dataclasses import dataclass
from typing import Literal

# engineering:context-resource-boundary
# Full match, ASCII: [a-z][a-z0-9]*(?:-[a-z0-9]+)*(?:/[a-z][a-z0-9]*(?:-[a-z0-9]+)*)*
# Case-sensitive; no normalization, percent-decoding, leading/trailing slash, or extension.
TopicPath = str


@dataclass(frozen=True)
class Topic:
    path: TopicPath                 # derived from the source's relative path, minus .md
    title: str                      # nonempty, single-line frontmatter value
    when_to_read: str               # nonempty, single-line frontmatter value
    body: str                       # authored Markdown after frontmatter, preserved verbatim
    links: tuple[TopicPath, ...]     # declared order; no duplicate or unresolved targets


@dataclass(frozen=True)
class ContextCatalog:
    topics: Mapping[TopicPath, Topic]      # generated records; immutable at runtime
    aliases: Mapping[TopicPath, TopicPath] # directory → directory/index; no leaf aliases


@dataclass(frozen=True)
class ContextLink:
    topic: TopicPath
    title: str
    when_to_read: str
    command: str                    # root: "blizzard context"; other indexes: directory alias
                                    # leaves: "blizzard context " + topic


@dataclass(frozen=True)
class ContextDocument:
    source: Literal["builtin"]
    cli_version: str
    topic: TopicPath                # canonical file-derived key, including /index where applicable
    title: str
    body: str                       # Topic.body for every page, including indexes
    links: tuple[ContextLink, ...]


@dataclass(frozen=True)
class CommandExample:                # example-surface validation only; not a runtime context client
    topic: TopicPath
    argv: tuple[str, ...]            # one marked command, parsed without a shell
    mode: Literal["syntax-only", "read-only-fixture"]
```

| Invariant                                                                                                                   | Invalid catalog behavior                                  |
| --------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| The authored root `index.md` exists.                                                                                        | Fail the build.                                           |
| Every path is unique and satisfies the full-match grammar.                                                                  | Fail the build.                                           |
| Every topic has valid metadata and a nonempty Markdown body.                                                                | Fail the build, naming the source and field.              |
| Every source directory has a readable `index.md`.                                                                           | Fail the build, naming the missing index.                 |
| A directory alias cannot collide with a leaf key: `work.md` and `work/index.md` are ambiguous. | Fail the build, naming both sources. |
| Every declared link resolves to a topic key or directory-index alias.                                                       | Fail the build, naming the source and target.             |
| Every topic is reachable through index navigation from the root; duplicate link targets are checked after alias resolution. | Fail the build on unreachable pages or duplicate targets. |
| Runtime catalog corruption never triggers source-file or service fallback.                                                  | Exit 1 before emitting a document.                        |

## Interfaces

### CLI

```text
blizzard context                     → index.md
blizzard context index               → index.md (identical response)
blizzard context work                → work/index.md
blizzard context work/index          → work/index.md (identical response)
blizzard context work/inspect        → work/inspect.md
blizzard context <path>              → one authored Markdown topic
blizzard context --help              → existing Click help behavior
```

| Input/outcome                               | Exit                  | stdout                                            | stderr                                                                  |
| ------------------------------------------- | --------------------- | ------------------------------------------------- | ----------------------------------------------------------------------- |
| Known topic or root                         | 0                     | Complete Markdown document, followed by a newline | Empty                                                                   |
| Invalid topic syntax or unknown path        | 2                     | Empty                                             | Name the input and say `Run blizzard context to list available topics.` |
| Invalid option or extra positional argument | 2                     | Empty                                             | Existing Click usage diagnostic                                         |
| Missing/corrupt generated catalog           | 1                     | Empty                                             | Identify the packaging error; no traceback or replacement fetch         |
| Output-stream failure                       | Existing CLI behavior | No success claim                                  | Existing CLI behavior                                                   |

```python
def resolve(requested: str | None, catalog: ContextCatalog) -> Topic:
    key = "index" if requested is None else requested
    validate_topic_path(key)  # reject invalid syntax; never normalize or decode
    canonical = catalog.aliases.get(key, key)
    if canonical not in catalog.topics:
        raise UnknownTopic(key)  # exit 2; no inferred directory or synthetic page
    return catalog.topics[canonical]
```

### Markdown

````markdown
<!-- source: builtin; cli_version: 0.5.0; topic: index -->

# Blizzard context

Read the branch that matches your task with `blizzard context <path>`.

```text
├── overview → When identifying Blizzard's components and your session's role
└── work → When choosing how to inspect, prepare, control, or respond to work
```
````

The abbreviated example body is authored in `topics/index.md`; `work/index.md` routes onward to its task leaves. Index
navigation order comes from its Markdown and declared links, not a global sort or automatic concatenation of descendant
documents. The complete source inventory follows `product:task-taxonomy` in [Markdown sources](#markdown-sources).

```text
index or leaf response = metadata comment + blank line + Topic.body
                        + optional "## Next topics" navigation + terminal newline

next-topic row = "- `" + ContextLink.command + "` → " + ContextLink.when_to_read
```

The authored body is unchanged, including headings, lists, and code fences. Metadata and navigation are generated
wrappers; a missing terminal newline is supplied by the renderer, not inserted into `ContextDocument.body`.

All title and read-trigger metadata comes from the selected Markdown source, including root and directory indexes.

### Internal boundary

```python
def compile_catalog(sources: Mapping[str, str]) -> ContextCatalog:
    """Build-time input: source-relative .md paths mapped to UTF-8 text."""

def read_context(
    topic: str | None, catalog: ContextCatalog, cli_version: str
) -> ContextDocument:
    """Pure projection of resolve(topic, catalog); aliases return identical documents."""

def render_markdown(document: ContextDocument) -> str: ...

# engineering:context-example-proof; test tooling only
def validate_example_surface(example: CommandExample, cli_surface: object) -> None:
    """Check exported command/argument/option metadata; never invoke command bodies or callbacks."""
```

## Data

### Markdown sources

```text
src/blizzard/cli/context/
├── topics/                         # human-authored source of truth
│   ├── connection.md
│   ├── delivery.md
│   ├── index.md                    # authored root hub
│   ├── overview.md
│   ├── runner-hosts.md
│   ├── work/
│   │   ├── index.md                # authored branch hub; alias: work
│   │   ├── control.md
│   │   ├── inspect.md
│   │   ├── prepare.md
│   │   └── respond.md
│   └── workers.md
└── _builtin.py                     # generated during distribution build; never hand-edited

topics/index.md           → index          # also selected when TOPIC is omitted
topics/work/index.md      → work/index     # alias: work
topics/work/inspect.md    → work/inspect    # explicit leaf; never a directory listing
```

```markdown
---
title: Overview
when_to_read: When identifying Blizzard's components and your session's role
links:
  - connection
---

# Overview

A hub coordinates work; runners execute leased steps.
```

| Source field            | Parsing contract                                                                                                                        |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| File                    | UTF-8 Markdown; `.md` extension; no symlink sources or paths escaping `topics/`.                                                        |
| Frontmatter             | One leading YAML mapping, parsed safely; strip only that block and its delimiter line endings.                                          |
| `title`, `when_to_read` | Required strings, nonempty after trimming, no newline characters; reject unknown metadata keys.                                         |
| `links`                 | Optional topic-key or directory-alias list, defaults to empty; resolve and reject canonical duplicates or unknown names.                |
| Body                    | Remaining Markdown text, including its original line endings and terminal newline; non-whitespace content required.                     |
| Path                    | Relative file path without `.md`, separators mapped to `/`; no filesystem path is accepted at runtime.                                  |
| Directory alias         | Each `<directory>/index.md` also accepts `<directory>`; omission selects root `index`. No alias may collide with another canonical key. |

| Source | Routing contract / command surfaces described |
| --- | --- |
| `index.md` | Routes to `overview`, `connection`, `work`, `delivery`, `workers`, and `runner-hosts`, with read-trigger cues. |
| `overview.md` | Hub, runner host, runner, project, chunk, and graph vocabulary; interactive versus leased roles; routes onward to connection and task guidance. |
| `connection.md` | Explicit destination and tenant/project scope, authentication, CLI-versus-hub version boundary, workspace binding/eager entry, and missing-setup recovery. |
| `work/index.md` | Routes to `work/inspect`, `work/prepare`, `work/control`, and `work/respond`; no combined operations manual. |
| `work/inspect.md` | Chunk status, current node, history, work items, artifacts, and evidence; routes a landing wait to `delivery`. |
| `work/prepare.md` | Ingest, group, true prerequisite links, order, promote, and queue operations; separates product mechanics from tenant policy. |
| `work/control.md` | Chunk pause/resume/stop and recovery controls, their scope, eligibility, and consequences. |
| `work/respond.md` | Read and answer worker questions and resolve human decisions within the current user's authority. |
| `delivery.md` | Landing policies, checks, mergeability, waits and failures, external merge recognition, and graph completion after landing. |
| `workers.md` | Leased node-step responsibilities and runner-facing artifact/ask/history interfaces; runtime identity and grants are supplied, never inferred. |
| `runner-hosts.md` | Remote runner availability versus machine-local host status and controls; host ownership, fleet-wide restart impact, and when to hand off. |

Navigation advertises only sources included in that build; expanding the corpus adds the corresponding index links in
the same change, without placeholder pages or dangling requests. Root and work indexes route to branches, not every
descendant's full content. Source content describes usage, not deployed identifiers or records.

### Packaging

```text
engineering:context-markdown-source

wheel build / editable build
  → enumerate authoritative Markdown sources
  → parse metadata + retain Markdown body
  → validate index presence, alias collisions, reachability, and links
  → emit _builtin.py from Topic records using deterministic UTF-8 serialization
  → include generated module in distribution

source distribution
  → include authoritative Markdown + compiler/build hook
  → wheel built from that source distribution regenerates the same topic payload

installed CLI
  → import generated catalog
  → no open(cwd / path), open(source_checkout / path), Markdown compilation, or network call
```

| Boundary             | Contract                                                                                                              |
| -------------------- | --------------------------------------------------------------------------------------------------------------------- |
| Content ownership    | Reuse an authoritative Markdown source or move its ownership; no hand-maintained application string copies.           |
| Generated payload    | Metadata and bodies match the parsed sources; rebuilding replaces all generated records, including removed topics.    |
| Runtime dependencies | Use existing Python/build facilities; no runtime Markdown-rendering dependency.                                       |
| Factory data         | Artifact scopes, graph content, runtime prompts, leases, and connection configuration keep their existing interfaces. |
| Persistence          | No database schema, migration, saved connection, or workspace-file write.                                             |

### Command examples

````markdown
<!-- context-example: syntax-only -->
```sh
blizzard hub chunk promote <chunk-id> --hub-url https://factory.example.invalid --tenant <tenant-id>
```
````

```text
marked single-command Markdown block
  → tokenize as argv without a shell; reject pipelines, redirections, substitutions, and command chaining
  → validate against the shipped CLI's exported command/argument/option metadata
  → syntax-only: never execute the command or its callbacks
  → read-only-fixture: execution permitted only by the controlled scenario's explicit command allowlist

topic requests in index navigation
  → resolve against the built catalog
  → execute only the context read, with network and worker/factory clients denied
```

| Marker / boundary | Contract |
| --- | --- |
| `context-example: syntax-only` | Validation is metadata-only, including for mutation examples; placeholders are sample argument values, not real identities. |
| `context-example: read-only-fixture` | Same surface validation; a separate controlled scenario supplies concrete fake values and an allowlisted read-only command. |
| Unmarked code fence | Illustrative text; not executable. The scenario cannot treat it as authorization to run a command. |
| Reused operator Markdown | Uses the same markers and validation; no separate example corpus maintained by copying its command text. |

### Workspace adoption example

The following two workspace-owned fragments are documented in `connection.md`; they are not additional context topics.

```markdown
<!-- Human-authored workspace context; not a Blizzard CLI configuration format. -->

# Factory participation

This workspace participates in Blizzard. Participation grants neither operator authority nor a fleet-worker assignment.
Use only the target declared below; do not infer production from a running local hub.

Read `context/project/factory.md`, then run `blizzard context` before factory interaction. If the binding or CLI is
missing, ask the operator to supply it; do not install, scan siblings, or substitute localhost. A leased worker takes
its assignment from its execution context.

<!-- Separate workspace-owned binding: context/project/factory.md -->

# Factory binding

| Fact                               | Workspace-owned value                                  |
| ---------------------------------- | ------------------------------------------------------ |
| Hub                                | https://factory.example.invalid                        |
| Tenant / project                   | <tenant-id> / <project-id-or-slug>                     |
| CLI availability                   | <operator-provided executable or supported invocation> |
| Machine responsibility             | Client only; no hub or runner-host administration here |
| Repository / work-source mappings  | <workspace-owned mappings or pointer>                  |
| Local approval / priority policies | <workspace-owned policy pointer>                       |
| Authentication                     | <secure sign-in instructions; no committed token>      |

Use `blizzard context connection` to apply the binding through existing CLI options. Use `blizzard context delivery` for
the route from chunk status to delivery evidence.
```

### Diagnosis walkthrough

```yaml
# Controlled scenario inputs, not a new user configuration format.
available:
  - installed blizzard wheel
  - workspace eager entry and binding from connection.md
  - authenticated fixture session for the binding's tenant and project
  - controlled hub reporting its own version and a fixture chunk's delivery evidence
absent:
  - Blizzard development checkout
  - runner or worker identity
  - other workspaces or sibling checkout discovery
route:
  - index
  - connection
  - work/inspect
  - delivery
factory_commands:
  - blizzard hub chunk get <fixture-chunk> --hub-url <fixture-hub> --tenant <fixture-tenant>
factory_request_boundary:
  destination: fixture-hub only
  methods: [GET]
  scope: fixture-tenant and the chunk belonging to fixture-project
context_request_boundary:
  factory_requests: none
missing_setup:
  binding: stop and request the workspace's factory facts
  cli: stop and request a supported invocation
forbidden_actions:
  - substitute localhost
  - install dependencies or start services
  - mutate work, connection configuration, or runner hosts
```
