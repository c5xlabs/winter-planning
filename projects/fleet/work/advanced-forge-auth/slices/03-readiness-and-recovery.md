---
name: readiness-and-recovery
depends-on:
  - github-app-installations
---

# Readiness and recovery

Delivers the read-only validation surface, known-capability checks before promotion into delivery, and the operator's
check, repair, and resume loop. Delivers `product:readiness-check` and `product:actionable-refusals`.

## Change

- A read-only integration check reports access and known gaps per operation.
- Work is checked for known incompatibilities before it is promoted into the affected delivery path.
- An escalated delivery resumes after the operator replaces and checks its credential.

## Acceptance

- Insufficient permissions and lost access leave an in-flight delivery safe.
- Each refusal names what the operator must change.
- A resumed delivery reuses its pull request and keeps its recorded landings.
