---
epic: preview
refinement: outline
status: planned
---

# Live preview

A standalone epic; its requirements are inline below.

## The problem

A reviewer reads a diff and imagines the feature. That is the whole of what blizzard offers today, and it is a poor
substitute for opening the thing and using it, particularly for work whose value is in how it feels. Meanwhile the
environment the chunk is working in is alive, running, and has the branch checked out. It is simply unreachable.

## The outcome

The operator opens the running application from the board and judges it the way a user would. The unsolved half is
reachability, and it is the epic's real content (`decision:preview/reachability`).

## Requirements

### Open the running application (product:open-preview)

A live chunk offers its running environment where the operator is already watching it, and opening it costs the trust
boundary between hub and runner nothing. A finished chunk does not offer what no longer exists, and a workspace that
hosts no application offers no preview rather than pretending.

Acceptance:

- From a live chunk on the board, the operator opens the application its environment is running.
- A finished chunk, or one whose workspace hosts nothing, offers no preview.

### A preview that dies with its chunk (product:preview-lifetime)

The environment's lifetime is the chunk's, and the platform is explicit about that rather than leaving a link that
quietly stops working.

Acceptance:

- When the chunk ends, its preview is shown as gone rather than left as a dead link.

## Open questions

- **Product owner:** What a provider that runs no services offers instead, if anything.
- **Product owner:** Whether a preview is ever shareable beyond the operator, which is a small feature and a large
  security question.
