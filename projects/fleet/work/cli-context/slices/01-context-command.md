---
name: context-command
code:
  - blizzard/src/blizzard/cli/main.py::blizzard
---

# Add bundled context navigation to the Blizzard CLI

Build `product:blizzard-usage-context`, `product:context-navigation`, and `product:installed-context` through the
catalog and CLI in [spec.md](../spec.md#interfaces), delivering `engineering:context-resource-boundary`,
`engineering:context-package-proof`, `engineering:context-usage-boundary`, and `engineering:context-markdown-source`.
Establish the initial `product:task-taxonomy` pages and workspace binding for `product:workspace-participation`, with
the reusable validation mechanism from `engineering:context-example-proof`.

## Change

- Register the additive top-level command with authored index navigation, Markdown output, package metadata, exact topic
  lookup, directory-index aliases, and actionable failure paths.
- Ship authored `index.md`, `overview.md`, and `connection.md`, deriving their content from
  [the Markdown source contracts](../spec.md#markdown-sources). Include the workspace-owned binding/eager-entry example
  and missing-setup behavior in connection guidance; advertise only these included pages, without future-topic stubs.
- Author guidance in readable `.md` files and include the documents in the existing Python wheel as build-generated
  embedded constants, per [packaging](../spec.md#packaging). Validate the catalog and update the CLI contract and change
  note; leave service behavior and connection configuration unchanged.
- Add the shared [command-example validation](../spec.md#command-examples) that later topic sources use, checking
  exported argument/option metadata without executing mutation bodies or callbacks.

## Acceptance

- Public-CLI tests pin omitted-topic/`index` equivalence and directory/explicit-index equivalence, known/unknown topics,
  invalid path inputs, Markdown output, and no partial document on failure.
- A nested-topic fixture serves the actual root and branch `index.md` bodies, including their authored read-trigger
  routing and order. Leaves return only their own content; each alias identifies the same canonical page, without
  synthesizing a tree or concatenating descendant bodies.
- A built-wheel test reads every included topic from an unrelated directory with networking denied, an empty home, no
  credentials, and no daemon; source/version metadata is correct and configuration locations remain unchanged.
- Source-to-payload tests compare installed topic bodies with their authoritative Markdown, including headings, lists,
  and code fences. Rebuilding after a source edit updates the packaged content, without maintaining application string
  literals or relying on checkout-relative runtime files.
- Foundation-page tests pin overview/connection coverage, including components and roles, explicit hub/tenant/project
  selection, secure authentication, the CLI/hub version boundary, and workspace binding/eager-entry setup recovery.
- Marked-example validation accepts the foundation's current command forms and rejects unknown commands/options;
  syntax-only examples cannot invoke any callbacks, execute a mutation, or contact a service.
- Isolation tests omit worker identity, reject artifact names outside the usage catalog, and fail on any attempted
  artifact, graph, worker, or prompt API call. Runtime data cannot affect context output, and existing artifact/prompt
  command contracts remain unchanged.
- Catalog tests reject duplicate addresses or alias/leaf collisions, missing resources or cues, missing directory
  indexes, unreachable topics, and dangling links. Advertised next requests execute successfully, and existing CLI
  compatibility checks remain green.
