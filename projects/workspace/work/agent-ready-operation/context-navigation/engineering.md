# Context navigation engineering requirements

For `epic:agent-ready-operation/context-navigation`.

## Version-bound framework context (engineering:version-bound-context)

Framework context is distributed with the Workspace component and addressed deterministically for that version; offline
reads cannot fetch newer framework content or fall back to a copied legacy framework tree. Serves
`product:progressive-context`.

## Read-only source resolution (engineering:context-source-safety)

Topic resolution enforces source boundaries, rejects path traversal and ambiguous source names, and does not execute
content or extension entry points. Missing or unreadable sources are reported without suppressing the distinction
between built-in, project, and extension context. Serves `product:unified-context-sources`.

## Consistent hook and router (engineering:safety-hook-consistency)

The eager hook and on-demand router carry the same essential safety obligations and have coverage for harnesses with and
without session-start execution. Common-hook updates follow
`product:parallel-workspaces/workspace-bootstrap/managed-content-ownership`, not whole-file replacement or a dependency
on later harness projection; harness-specific projection consumes that baseline. Serves
`product:eager-workspace-safety`.
