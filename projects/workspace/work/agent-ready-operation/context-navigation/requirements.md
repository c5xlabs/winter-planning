# Workspace context requirements

For `epic:agent-ready-operation/context-navigation`.

## Navigate only the guidance needed (product:progressive-context)

An agent can start with a context root and request the next topic by the address shown there. Guidance comes from the
selected component rather than a separately copied framework tree.

Acceptance:

- The root describes the relevant branches and each index gives runnable next-topic requests.
- Requesting a topic returns that topic, not the entire framework documentation set.
- Framework help matches the invoked Workspace version and remains available without a network connection.
- Unknown topics produce an actionable error rather than selecting a similarly named file.

## Reach project and extension context (product:unified-context-sources)

The engineer can use one context front door for framework, project-owned, and installed extension guidance without
memorizing their physical paths.

Acceptance:

- The response identifies its source so similarly named topics cannot be confused.
- Project-extension guidance follows the selected feature worktree when an environment is bound; any source-checkout
  fallback is explicit rather than silently injecting another branch's instructions.
- Installed context-only extensions remain usable without executable plugins.
- Raw text and machine-readable consumption are available without requiring a TUI.
- Reading context does not install an extension, run its hooks, or rewrite the workspace.

## Read safety rules before acting (product:eager-workspace-safety)

An agent sees the provision-before-run and destructive-scope rules even when its harness cannot run a command during
context loading. The delivery choice is owned by `decision:agent-ready-operation/context-navigation/safety-hook`.

Acceptance:

- A newly initialized or reconciled workspace exposes the essential rules through its supported harness entry points.
- An agent without session-start command hooks still receives an eager text entry point and a way to request details.
- Project-owned instructions remain intact when the generated Workspace hook is updated.
