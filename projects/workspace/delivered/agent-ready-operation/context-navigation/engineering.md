# Context navigation engineering requirements

For `epic:agent-ready-operation/context-navigation`.

## Version-bound framework context (engineering:version-bound-context)

Every topic is embedded in the Workspace binary at build time, and the topic map is fixed per build: the same binary
answers the same id with the same bytes in every run. Serves `product:progressive-context`.

## Small entry point (engineering:entry-size)

The body of the eager safety text is at most 15 lines and the served text of the root is at most 80 lines; a build that
exceeds either fails. Serves `product:progressive-context` and `product:eager-workspace-safety`.

## Built-in topic resolution (engineering:built-in-topics)

A topic id resolves only against the topic map compiled into the binary. Resolution reads no workspace file, so an id
cannot reach a path outside the built-in set. An unknown id exits non-zero, names the id, and names the request that
lists the available topics; it never selects the closest match. A test checks that every id an index lists resolves.
Serves `product:progressive-context`.

## One source for the safety text (engineering:safety-text-consistency)

The eager safety text and the context root carry the same safety rules from one source in the binary, and a test fails
when they differ. The binary exposes the eager safety text for `epic:parallel-workspaces/workspace-bootstrap` to place;
harness-specific projection consumes the same text. Serves `product:eager-workspace-safety`.
