---
name: readiness-and-recovery
external: github:paul-gross/blizzard#2183
depends-on:
  - app-installation-binding
  - credential-document-and-board
  - mock-forge
  - refusal-diagnostics
code:
  - blizzard/src/blizzard/hub/delivery/actions/landing/repo.py::RepoLanding
---

# Readiness and recovery

Adds the read-only credential check at every door and the recovery step in a `forge-auth` escalation, and proves the
recovery loop. Delivers `product:readiness-check` and the recovery half of `product:actionable-refusals`. Builds
readiness-check.md's [Domain](../spec/readiness-check.md#domain) and check interfaces, and the detail's `Then run:` line
in [refusals.md](../spec/refusals.md#interfaces). Shaped by `decision:advanced-forge-auth/readiness-not-kept` and
`decision:advanced-forge-auth/recover-with-requeue`. Leaves out a claim-time gate, per
`decision:advanced-forge-auth/claim-time-gate`.

## Change

- `hub/forge/readiness.py`, `POST /api/credentials/{credential}/checks` with its write-refusing transport and
  `credential-check` event, `blizzard hub credential check`, and the board's Check action on credential, repository, and
  work-source details.
- The `forge-auth` detail ends with the check and requeue commands for the chunk.
- `docs/deployment/credentials.md` gains the check and its outcomes; `recovery.md` gains fix, check, requeue.

## Acceptance

- The transport raises on every method but `GET` and the App mint's `POST`, and the mock forge records no write during a
  check.
- Against the mock forge's levers, the check answers each of `proven`, `granted`, `unproven`, `refused`, and
  `unsupported` for its case, states that branch protection is not evaluated, and stops at 60 s reporting unreached
  uses; a use not naming the credential is refused `422 credential_not_used_by`.
- The CLI exits 1 when any outcome is `refused` or `unsupported` and 0 otherwise; a board component test renders each
  outcome.
- For each refusal: after fixing the credential, checking it, and requeueing, the delivery merges the same change and
  records each repository's landing once.
- A permission withdrawn mid-delivery escalates with the change open, its branch present, and earlier landings kept.
