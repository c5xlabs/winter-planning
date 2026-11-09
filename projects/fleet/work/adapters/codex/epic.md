---
epic: adapters/codex
refinement: refined
status: planned
---

# Codex

Part of `initiative:adapters`. Its product requirements are not yet written; they will follow the shape
`epic:adapters/opencode` set: prove the real CLI, make a usable worker, reach conversation parity, and harden it for
unattended work.

## The problem

Codex is the third harness behind the adapter seam. A model offered only through Codex stays outside the fleet until a
binding brings it in.

## The outcome

Codex follows the same contract OpenCode establishes, and it waits for OpenCode deliberately: only after two occupants
have forced every Claude-shaped assumption into the binding that owns it is the seam honest enough for a third to arrive
as a binding and nothing more. Codex is a new adapter under harness selection that already exists, not a new round of
platform change.
