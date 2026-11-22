# Context navigation engineering requirements

For `epic:agent-ready-operation/context-navigation`.

## Version-bound framework context (engineering:version-bound-context)

Framework context is compiled into the Workspace binary and addressed deterministically for that version. Offline reads
cannot fetch newer framework content or fall back to a copied legacy framework tree. Serves
`product:progressive-context`.

## Built-in topic resolution (engineering:built-in-topics)

A topic id resolves only against the topic map compiled into the binary. Resolution reads no workspace file, so an id
cannot reach a path outside the built-in set. An unknown id exits non-zero, names the id, and names the request that
lists the available topics; it never selects the closest match. A test checks that every id an index lists resolves.
Serves `product:progressive-context`.

## Consistent hook and router (engineering:safety-hook-consistency)

The eager safety text and the context root carry the same safety rules from one source in the binary, and a test fails
when they differ. The binary exposes the eager safety text for `epic:parallel-workspaces/workspace-bootstrap` to place;
harness-specific projection consumes the same text. Serves `product:eager-workspace-safety`.
