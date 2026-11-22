# Context navigation spec

How the framework guidance is written, compiled into the binary, and served by the `context` command, and how the eager
safety text reaches `epic:parallel-workspaces/workspace-bootstrap`. Command registration and the path rules of
`commands/` are those of
[command-registration.md](../../../delivered/parallel-workspaces/component-runtime/spec/command-registration.md); the
package layout is that of
[repository-layout.md](../../../delivered/parallel-workspaces/component-runtime/spec/repository-layout.md); streams and
JSON shape A are those of
[output-formats.md](../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md); `Termination` and
`CommandError` are those of
[results-and-exit-codes.md](../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
Domain. The contributor tree `context/` of
[contributor-context.md](../../../delivered/parallel-workspaces/component-runtime/spec/contributor-context.md) is a
separate tree for contributors and is not served by this command.

## System

```text
# engineering:version-bound-context, decision:agent-ready-operation/context-navigation/topic-addressing
crates/core/topics/**/*.md        topic sources; the directory layout is the author's and is never an address
crates/core/build/topics.rs       the topic rules and the code generation, as pure functions build.rs includes;
                                  under cfg(test) the library includes it too, so it is unit-tested
crates/core/build.rs              also walks topics/ and writes $OUT_DIR/topics.rs
crates/core/src/topics.rs         include!(concat!(env!("OUT_DIR"), "/topics.rs")); lookup and the eager text
crates/core/src/commands/context.rs   the `context` command: one call into topics, one result written

rebuild    build.rs prints cargo:rerun-if-changed for crates/core/topics and every directory under it
no reads   at run time the command reads no file, no environment variable, and no workspace: it needs no workspace
           root and runs the same in any readable working directory (engineering:built-in-topics)
```

```text
checked by crates/dev/tests/architecture.rs over crates/core/src/topics.rs and crates/core/src/commands/context.rs
  names none of   ReadFiles  WriteFiles  OsFiles  workspace_root  workspace_files  config_raw  std::fs  std::env
```

## Domain

```rust
// crates/core/src/topics.rs — engineering:built-in-topics
// A topic id: one or more segments joined by "/", each [a-z][a-z0-9-]*, as in "ws/push". An id names a topic, not
// a file: moving a source file under topics/ changes no id.
pub struct TopicId(&'static str);

pub struct Topic {
    pub id: TopicId,
    pub title: &'static str,          // one line, from the source's front matter
    pub text: &'static str,           // the served text: the source's body, unchanged; for root, Composition below
    pub next: &'static [TopicId],     // ids the text references, in order of first reference, without repeats;
                                      // never the topic's own id
}

// Generated. Sorted by id; one entry per authored topic plus the generated `topics`.
pub(crate) static TOPICS: &[Topic];

pub(crate) fn root() -> &'static Topic;                          // id "root"; what `context` with no TOPIC serves
pub(crate) fn find(id: &str) -> Result<&'static Topic, ContextError>;   // exact match on the whole string only
pub fn eager_text() -> &'static str;                             // the body of `safety`; for workspace-bootstrap

// crates/core/src/lib.rs re-exports eager_text and ContextError; Topic, TopicId, TOPICS, root, and find stay
// crate-internal (repository-layout.md, What stays a shared edit)

#[derive(Debug, thiserror::Error)]
pub enum ContextError {
    // Any string that is not the id of a topic, including one that is not a valid id ("../x", "WS/push", "").
    // Display names the id and the request `winter context topics`. No closest-match suggestion.
    UnknownTopic { id: String },
}
// CommandError gains: #[error(transparent)] Context(#[from] ContextError)   — exit 1
```

### Reserved ids

| Id       | Source                    | Content                                                                                       |
| -------- | ------------------------- | --------------------------------------------------------------------------------------------- |
| `root`   | authored, required        | The entry point: what the guidance covers and a request for each branch.                      |
| `safety` | authored, required        | The eager safety text, whole. Composed into `root`; returned by `eager_text()`.               |
| `topics` | generated, never authored | One line per topic, itself included, sorted by id: `` `winter context <id>` `` and its title. |

### Composition

```text
# engineering:safety-text-consistency — one source for the safety rules
served text of root   = body of safety + "\n" + body of root
eager_text()          = body of safety, byte for byte
a test asserts        root().text starts with eager_text(), and eager_text() is the body of the source whose id is safety
```

## Interfaces

### Command

| Command   | Arguments | Options  | `--json` | Purpose                                                           |
| --------- | --------- | -------- | -------- | ----------------------------------------------------------------- |
| `context` | `[TOPIC]` | `--json` | A        | Print one topic of this version's guidance; no TOPIC prints root. |

```text
winter context                 root: the safety text, then the entry point
winter context ws/push         one topic
winter context topics          every id with its title
winter context <TOPIC> --json  the same topic as one shape A document
```

```text
stdout, human     the title on one line, a blank line, then the topic's text exactly as served, then a line break
                  if the text does not end with one
stdout, --json    {"id": <string>, "title": <string>, "text": <string>, "next": [<string>, ...]}
                  "text" is Topic.text byte for byte and is not message text; "next" is Topic.next, in order
unknown TOPIC     Termination::Command(CommandError::Context(UnknownTopic)); stderr only, stdout empty; exit 1
                  same with --json: nothing on stdout
success           exit 0
```

### References from other documents

```text
# decision:agent-ready-operation/context-navigation/product-scope
winter:/workspace            the Workspace root; read with `winter context`
winter:/workspace/<id>       a Workspace topic; read with `winter context <id>`
the `workspace` segment names the product and is never part of an id: the binary refuses `winter context workspace/x`
as UnknownTopic unless a topic's own id is `workspace/x`
```

### Eager text for bootstrap

```rust
// engineering:safety-text-consistency
// workspace-bootstrap places this text in AGENTS.md inside the managed block it owns
// (product:parallel-workspaces/workspace-bootstrap/managed-content-ownership). It never edits the text and adds
// nothing inside it; the block's markers are bootstrap's.
// It is the only topic text placed in any file: root, setup, and every other topic are read through the command.
pub fn eager_text() -> &'static str;
```

The eager safety text, the source whose id is `safety`:

```markdown
---
id: safety
title: Workspace safety rules
---

## Workspace safety

- Bring a feature environment up in order: `winter ws init <env>`, then `winter provision <env>`, then
  `winter service up <env>`. Never run or exercise an environment that has not been provisioned.
- Never run a destructive command against a live environment you do not intend to change: `winter ws checkout`,
  `winter ws reset --hard`, `winter ws clean`, `winter ws destroy`, `winter ws restack`, `winter clean`, and
  `winter provision --destroy`. Each reaches every worktree or environment its arguments match. Try one in a scratch
  environment first.
- `winter context` prints this Workspace version's guidance and `winter context topics` lists every topic. Other
  documents cite a topic as `winter:/workspace/<topic>`.
```

## Data

### Topic source

```markdown
---
id: ws/push
title: Push matched worktrees to their upstream
---

Body: Markdown, served unchanged. A reference to another topic is the code span `winter context <id>`.
```

```text
# crates/core/build/topics.rs — a violation fails the build with a message that starts with the offending path
files         every entry under topics/ is a directory or a *.md file
front matter  the file starts with exactly: "---\n", "id: <id>\n", "title: <one line>\n", "---\n"
id            matches the TopicId grammar; unique across all files; not "topics"
required      ids "root" and "safety" exist
body          everything after the front matter's closing line, unchanged
reference     an inline code span outside any fenced or indented code block whose whole text is `winter context`
              (root) or `winter context ` followed by a string matching the TopicId grammar
refused       an inline code span outside code blocks whose text is `winter context ` followed by a TopicId and then
              more text, as in `winter context ws/push --json`: a link must be the bare request
not a link    a span such as `winter context <topic>`, whose rest matches no TopicId; anything inside a code block
resolves      every reference names an authored id, "root", or "topics"
reachable     every authored topic except safety is reached from root by following references
size          engineering:entry-size: safety body at most 15 lines; served text of root at most 80 lines
```

### Coverage of commands

```text
a component test over REGISTRY, so a change that adds a command fails until it adds the command's topic
  every Registration::Command entry that is not hidden   has a topic whose id is its path joined by "/"   ws/index
  every group that holds such an entry                   has a topic whose id is the group's name         ws
  an Alias entry                                          needs no topic; its sibling's topic covers it

held in review, not by a test (product:framework-guidance)
  a topic's text describes only commands and options of the build it ships in
  safety is exempt: it states the workspace rules for every Workspace command, including ones the build lacks

added by this epic     root, safety, context, topics, and one topic per command and group the build holds at delivery
added later            a topic for each command or workspace concept, by the change that adds it
```

### Setup topics

```text
# decision:agent-ready-operation/context-navigation/setup-guidance
setup                         branch index; root references it; lists only the leaves the build holds
                                                                                          epic:parallel-workspaces/workspace-bootstrap
setup/walkthrough             the interactive, re-runnable workspace setup procedure      epic:parallel-workspaces/workspace-bootstrap
setup/contributing            writing a workspace's context/project/contributing.md       epic:parallel-workspaces/workspace-bootstrap
setup/project-setup           writing a workspace's context/project/project-setup.md      epic:parallel-workspaces/workspace-bootstrap
setup/service-discovery       what the walkthrough records about an application's services
                                                                                          epic:running-environments/service-orchestration
setup/service-orchestration   choosing and configuring a service provider                 epic:running-environments/service-orchestration
each topic is added by the epic named beside it, with the commands it walks through; setup and its root reference
land with the first leaf, so setup is never an empty index
no setup topic is a skill or is placed in AGENTS.md; an agent reaches it from root, or a person asks for it by id
```
