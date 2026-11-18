---
name: forge-auth-harness
external: github:paul-gross/blizzard#2184
depends-on:
  - refusal-diagnostics
  - authorization-boundary
  - readiness-and-recovery
---

# Forge authentication harness

Teaches `blizzard-context` the credential, the boundary, the new events, and the recovery loop, so agents building on
the epic find them.

## Change

- `domain/operations.md` gains `forge-auth-refused` and `credential-check`; `domain/projects.md` and
  `architecture/system-shape/configuration.md` learn the credential; `architecture/system-shape/seam-positions.md` gains
  the authorization boundary; `domain/humans/escalation.md`'s `forge-auth` row names the diagnostic and the recovery.

## Acceptance

- `blizzard-context`'s link and reference validation passes.
