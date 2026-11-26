---
name: context-command
external: github:paul-gross/winter#238
depends-on:
  - topic-map
code:
  - winter-workspace/crates/core/src/termination.rs::CommandError
---

# `context` prints one topic of the compiled guidance

Adds `context [TOPIC] [--json]`, which looks a topic up in the compiled map and prints it, and the `context` topic that
describes it. Builds the lookup of [spec.md](../spec.md), Domain, and its Command and References from other documents
sections; output streams and JSON shape A are those of
[output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md), and the error
is a `CommandError` as
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
Domain, gives it. Delivers `product:progressive-context` and `engineering:built-in-topics`, under
`decision:agent-ready-operation/context-navigation/product-scope`.

## Change

- `crates/core/src/topics.rs` gains `root()`, `find()`, which matches the whole string exactly, and `ContextError` with
  `UnknownTopic`, whose message names the id and `winter-workspace context topics` and suggests no other topic.
  `CommandError` gains `Context`, exit 1; `lib.rs` re-exports `ContextError`.
- `crates/core/src/commands/context.rs` registers the command with the argument and option spec.md, Command, gives it,
  calls `topics::find`, or `topics::root` with no TOPIC, and writes one result: the title, a blank line, and the text as
  served, then a line break if the text does not end with one; with `--json`, the shape A document
  `{"id", "title", "text", "next"}`, with `text` byte for byte. An unknown topic ends as
  `Termination::Command(CommandError::Context(..))`.
- The architecture rule over `topics.rs` also covers `crates/core/src/commands/context.rs`.
- `crates/core/topics/context.md` describes the command and the `winter:/workspace/<id>` form of a reference, and `root`
  references it.

## Acceptance

- Component tests: with no TOPIC the command prints the title of `root`, a blank line, and the served text, which begins
  with `eager_text()`; `context topics` prints the generated listing; `context context` prints only that topic; a topic
  whose text ends with a line break prints no second one.
- A component test of `--json`: one shape A document whose `text` equals the topic's served text byte for byte and whose
  `next` is its references in order.
- Component tests of refusal: `Context`, `ROOT`, `context/`, `context`, `../x`, an empty string, and `workspace/context`
  each end with `CommandError::Context`, nothing on stdout, and a message naming the id and
  `winter-workspace context topics`; with `--json`, stdout is still empty.
- A component test: `context` run with the working directory in a fixture workspace whose `.winter/config.toml` does not
  parse exits 0 with the same stdout as outside a workspace, and no file of the fixture changes.
- A unit test: every id in every topic's `next` resolves through `find`.
- An executable test: `winter-workspace context` run in a directory with no workspace above it, with the workspace
  variables unset, exits 0 and prints the root; an unknown topic exits 1 with the message on stderr only.
- The architecture test fails on a `context.rs` that names `std::env::temp_dir`, which no earlier rule refuses, and
  passes on the tree.
- `cargo dev gate slice` exits 0.

## Out of scope

- Topics for the other commands and the coverage test: `command-topics`.
