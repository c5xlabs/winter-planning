---
epic: agent-ready-operation/context-quality
refinement: outline
status: planned
---

# Check the guidance and declarations being changed

Part of `initiative:agent-ready-operation`; product requirements live in `requirements.md`.

## The problem

An author can make guidance look correct inside one workspace while leaving undeclared dependencies, oversized injected
context, or invalid service references for the next consumer. A check that reads the wrong checkout or hides too much
under an ignore is worse than an actionable failure because the author trusts the clean result.

## The outcome

Authors and agents can check a declared content scope, including just dirty and unpushed work, with an identifiable
catalog of checks and audited exemptions. The result distinguishes content findings, failed checks, and incomplete
evidence without pretending the workspace was repaired.

## Depends on

- `epic:parallel-workspaces/workspace-configuration`
- `epic:parallel-workspaces/repository-catalog`
- `epic:agent-ready-operation/dependency-graph` for module dependency checks.
- `epic:agent-ready-operation/context-navigation` for identifying the guidance source being checked.

Contributed checks integrate with `epic:agent-ready-operation/executable-extensions`; service-reference checking
consumes `epic:running-environments/service-orchestration` catalog evidence without requiring services to be started or
provisioned.
