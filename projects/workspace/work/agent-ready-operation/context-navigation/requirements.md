# Workspace context requirements

For `epic:agent-ready-operation/context-navigation`.

## Navigate only the guidance needed (product:progressive-context)

An agent can start with a context root and request the next topic by the id shown there. Guidance comes from the
selected component rather than a separately copied framework tree. Topics are addressed under
`decision:agent-ready-operation/context-navigation/topic-addressing`.

Acceptance:

- The root describes the relevant branches and each index gives runnable next-topic requests.
- Requesting a topic returns that topic, not the entire framework documentation set.
- Framework help matches the invoked Workspace version and remains available without a network connection.
- Unknown topics produce an actionable error rather than selecting a similarly named topic.
- Raw text and machine-readable output are available without requiring a TUI.
- Reading context does not read or rewrite the workspace, install an extension, or run hooks.

## Find guidance for each command and for setup (product:framework-guidance)

The engineer or agent finds guidance for every Workspace command and for setting up a workspace through the context,
without a copied framework tree or a separate setup skill. Setup is settled by
`decision:agent-ready-operation/context-navigation/setup-guidance`.

Acceptance:

- Every visible command and command group of the invoked Workspace build has a topic.
- A topic describes only commands and options that build holds; the eager safety text alone may name commands the build
  lacks.
- Workspace setup is a branch of topics reachable from the root, not a skill, and is never placed in `AGENTS.md`.

## Read safety rules before acting (product:eager-workspace-safety)

An agent sees the provision-before-run and destructive-scope rules even when its harness cannot run a command during
context loading. The delivery choice is owned by `decision:agent-ready-operation/context-navigation/safety-hook`.

Acceptance:

- The eager safety text states both rules and how to request their details, and needs no command run to be read.
- The eager safety text and the rules the context root serves are the same text for the same Workspace version.
- `epic:parallel-workspaces/workspace-bootstrap` can place and update the eager safety text without editing it.
