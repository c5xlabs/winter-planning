# CLI context requirements

For `epic:cli-context`.

## Learn how to use Blizzard (product:blizzard-usage-context)

Context answers "How do I use Blizzard?" for people and agents. It explains the product and its operating commands,
rather than serving factory artifacts, configured prompts, or live work data through another interface.

Acceptance:

- The root presents usage guidance; topics explain when and how to use Blizzard's existing command surfaces, including
  where to retrieve factory data when a task needs it.
- Artifact, graph, prompt, and worker-assignment content remains behind its existing interfaces. Context may explain
  those interfaces but does not enumerate or return their live records, resolve their artifact names, or display a
  runtime's effective prompt.
- The guidance is useful to an interactive user without a worker lease or runner identity; being agent-readable does not
  make a context request a worker operation.

## Find the next useful topic (product:context-navigation)

A person or agent runs `blizzard context` to read the root `index.md`, then follows its topic paths and "when to read"
cues. Directory addresses open their own `index.md`, following the same progressive-disclosure convention as context
repositories rather than substituting a generated catalog for an authored page.

Acceptance:

- With no arguments, the command serves the authored root `index.md`, identically to `blizzard context index`.
- A directory address such as `work` serves `work/index.md`, identically to `work/index`; an explicit leaf address such
  as `work/inspect` serves that leaf only.
- Index pages provide a directory-style map of their branches, pairing runnable topic addresses with task-oriented "when
  to read" cues. They route to further indexes or leaves without loading their bodies or dumping the whole corpus.
- Topic pages provide runnable next requests and their "when to read" cues where the task needs another topic.
- `blizzard context <topic>` returns the requested topic, not an aggregate of the whole documentation tree.
- Unknown topics are refused with a way to discover valid ones, without substituting a similarly named topic.
- Index and leaf text is served as Markdown without a TUI or pager. Directory aliases and explicit index addresses
  identify the same page and preserve its authored body, read-trigger cues, and topic navigation.

## Choose guidance by task (product:task-taxonomy)

The context is organized around the questions a person or agent brings to Blizzard, rather than mirroring command
groups or collecting every interactive operation under an "operator" topic. Workspace participation and binding
guidance belongs with connection setup, not in a separate workspace topic.

Acceptance:

- The root routes to `overview` for components and roles, `connection` for hub/tenant/project setup, `work` for work
  operations, `delivery` for landing and delivery waits, `workers` for leased-step usage, and `runner-hosts` for runner
  availability and host controls.
- The `work` index routes to `work/inspect` for progress/history/evidence, `work/prepare` for ingestion, grouping, ordering,
  and promotion, `work/control` for pause/resume/stop/recovery, and `work/respond` for worker questions and human decisions.
- Each index is a small task router with read-trigger cues. Each leaf answers its named question and points to another
  topic rather than absorbing unrelated usage guidance.
- `connection` includes the workspace-owned binding and eager-entry examples; `runner-hosts` distinguishes inspecting
  remote availability from administering a host on the machine responsible for it.

## Read the installed version's guidance (product:installed-context)

The guidance arrives with the installed CLI and remains readable without a network, sign-in, configured workspace, or
running hub. A reader can tell which CLI version the guidance describes, rather than assuming it describes a remote hub
of a different version.

Acceptance:

- A clean installation can read the root and every bundled topic offline, with no application source checkout.
- Output identifies the document's source and CLI version; it does not report an unqueried hub's version or
  capabilities.
- Reading context does not authenticate, select a tenant, contact a service, install a dependency, or modify user or
  workspace state.

## Operate within the session's role (product:factory-operation)

The guidance distinguishes a person working with an interactive assistant from a fleet worker executing a leased step.
It explains how to find evidence before changing work and where an operation requires a different role or machine.

Acceptance:

- A reader can navigate to guidance for inspecting chunk progress and delivery evidence, answering a worker's question,
  and preparing, grouping, ordering, ingesting, and promoting work.
- The guidance distinguishes a hub from runner hosts and runners, and explains that a client machine need not host
  either; a local development hub is not evidence of a production connection.
- Participation in a factory does not identify a session as a fleet worker or grant it operator authority. A worker's
  chunk, node, lease, and granted actions come from its execution context, not inferred workspace membership.
- Read-only diagnosis, mutating work controls, and host administration are distinguished. Runner-host restarts warn of
  their impact on other running workers; a manual forge merge is distinguished from completing the whole graph.
- Commands and examples agree with the shipped CLI. Workflow advice separates Blizzard's mechanics from tenant-owned
  priority, approval, and delivery policies, and points to the appropriate live status rather than inventing a result.

## Supply the workspace's own factory facts (product:workspace-participation)

A workspace records where its factory is and how it participates, while Blizzard owns the generic operating guidance. A
minimal eager instruction leads an unfamiliar agent from those local facts into the CLI's context without searching
unrelated checkouts.

Acceptance:

- A documented, non-secret binding example covers the hub URL, tenant and project identity, CLI availability, and this
  machine's responsibilities. It identifies local work-source/repository mappings and policies as workspace-owned.
- A minimal agent-entry example states factory participation, points to the workspace-owned binding, and requests the
  CLI context before factory interaction; essential role and production-target cautions are present before that read.
- Missing binding or CLI availability leads to an actionable request for setup, not a localhost guess, sibling-directory
  search, fabricated credential, or automatic installation. Examples do not require Winter-specific integration.
- The binding is distinct from runtime worker facts and from credentials. Examples reference secure authentication setup
  rather than placing tokens in committed instructions, and do not claim the CLI consumes a new binding format.
- Given only the installed CLI, a fixture workspace's binding, and its agent-entry instruction, a reader can follow the
  documented route to the correct project-scoped inspection commands and delivery troubleshooting without a Blizzard
  development checkout.
