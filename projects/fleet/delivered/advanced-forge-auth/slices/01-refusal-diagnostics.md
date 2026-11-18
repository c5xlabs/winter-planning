---
name: refusal-diagnostics
external: github:paul-gross/blizzard#2174
depends-on:
  - epic:advanced-delivery/forges
code:
  - blizzard/src/blizzard/hub/forge/landing.py::ForgeRejected
  - blizzard/src/blizzard/hub/forge/internal/github_landing.py::GitHubLanding
  - blizzard/src/blizzard/hub/delivery/actions/landing/repo.py::RepoLanding
  - blizzard/src/blizzard/hub/work_sources/internal/github_work_source.py::GitHubWorkSource
  - blizzard/src/blizzard/hub/work_sources/source.py::WorkSourceError
  - blizzard/src/blizzard/hub/forge/commit_check.py::SeamCommitResolver
  - blizzard/src/blizzard/foundation/event_log.py::EventLogKind
---

# Refusal diagnostics

Classifies the forge's 401 and 403 answers and carries a sanitized diagnostic into the `forge-auth` escalation, a new
event, and work-source errors, before any credential exists. Delivers the refusal half of `product:actionable-refusals`
and `engineering:secrets-out-of-evidence`. Builds [refusals.md](../spec/refusals.md) except its `token-unsupported` row,
its mint rows, and the detail's `Then run:` line, which lands with the check command in `readiness-and-recovery`.

## Change

- `AuthRefusal`, `ForgeDiagnostic`, `ForgeAuthRefused`, and `WorkSourceAuthRefused`; `ForgeRejected` carries a
  diagnostic for its policy refusals.
- `classify_auth_refusal` and `sanitize_forge_text`, used by `GitHubLanding` and `GitHubWorkSource`.
- The `forge-auth` detail names the operation, status, GitHub's message, request id, and the repository's secret, with
  its remedy.
- The `forge-auth-refused` event, recorded per escalation and per refused sweep; `SeamCommitResolver` logs a refusal's
  diagnostic at `WARNING`.
- `docs/deployment/repositories.md` stops claiming a fine-grained token reads checks; `observability.md` names the
  event.
- Leaves alone: every escalation cause and its routing, and `rerun_check`'s refusal handling.

## Acceptance

- Each classifier row this slice builds maps its answer, one unit test per row.
- The sanitizer's property test finds no token, JWT, PEM block, or `Authorization` value in its output.
- Under the mock forge's existing `TOKEN_REJECTED` lever, a landing escalates `forge-auth` with the operation, status,
  message, and request id in its detail and records one `forge-auth-refused` event; a work source's sweep records the
  event naming the work source.
- No escalation, event, log line, or API answer in the suite carries the test token.
- A refused, unreadable, or empty check list never merges.
