---
name: readiness-and-recovery
depends-on:
  - github-app-installations
code:
  - blizzard/src/blizzard/hub/delivery/flow_turn.py::TurnRunner
  - blizzard/src/blizzard/hub/domain/config/authoring.py::ConfigAuthoring
  - blizzard/src/blizzard/hub/delivery/actions/landing/repo.py::RepoLanding
---

# Readiness and recovery

Adds the read-only credential check, the credential-edit and hub-start gates, and the recovery steps in a `forge-auth`
escalation. Delivers `product:readiness-check` and the recovery half of `product:actionable-refusals`. Builds
[readiness-check.md](../spec/readiness-check.md) apart from the gates the first slice built, and the recovery remedy in
[refusals.md](../spec/refusals.md#interfaces). Shaped by `decision:advanced-forge-auth/readiness-not-kept` and
`decision:advanced-forge-auth/recover-with-requeue`; leaves `decision:advanced-forge-auth/claim-time-gate` open.

## Change

- `POST /api/credentials/{credential}/checks`, `blizzard hub credential check`, and the board's Check action on
  credential, repository, and work-source details.
- Editing a credential re-checks its enabled referrers; hub start warns about undeclared fine-grained tokens behind a
  policy they cannot serve.
- A `forge-auth` escalation's detail, and the board's escalation panel for `forge-auth` and `forge-capability-missing`,
  give the check and requeue commands for the chunk.
- Docs: `docs/deployment/recovery.md` gains the fix, check, requeue loop; `credentials.md` gains the check and its
  outcomes; `blizzard-context`'s `domain/operations.md` gains `credential-check`.

## Acceptance

- The check's transport raises on every method but `GET` and the App mint's `POST`, and no forge write reaches the mock
  forge during a check.
- Against the mock forge's levers, the check answers each of `proven`, `granted`, `unproven`, `refused`, and
  `unsupported` for the case that should produce it, states that branch protection is not evaluated, and stops at 60 s
  reporting unreached uses.
- Editing a credential to `fine-grained` behind a `merge-on-checks` referrer is refused, listing the referrer.
- Hub start logs one warning per undeclared credential whose token is fine-grained behind a checks-reading policy.
- For each refusal: after fixing the credential, checking it, and requeueing, the delivery merges the same change and
  records each repository's landing once.
- A permission withdrawn mid-delivery escalates with the change still open, its branch present, and earlier landings
  kept.
