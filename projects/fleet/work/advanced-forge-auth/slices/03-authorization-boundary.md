---
name: authorization-boundary
external: github:paul-gross/blizzard#2176
depends-on:
  - credential-record
  - refusal-diagnostics
code:
  - blizzard/src/blizzard/hub/forge/registry.py::ForgeRegistry
  - blizzard/src/blizzard/hub/forge/registry.py::ForgeBinding
  - blizzard/src/blizzard/hub/forge/registry.py::WriteLocks
  - blizzard/src/blizzard/hub/forge/internal/github_landing.py::GitHubLanding
  - blizzard/src/blizzard/hub/work_sources/internal/factory.py::WorkSourceEntry
  - blizzard/src/blizzard/hub/forge/budget.py::BudgetKey
  - blizzard/src/blizzard/hub/forge/budget_notices.py::BudgetNoticeDrain
  - blizzard/src/blizzard/hub/live_config/__init__.py::CacheKey
  - blizzard/src/blizzard/hub/delivery/flow_turn.py::TurnRunner
---

# Authorization boundary

Puts landing and ingestion behind one authorization boundary with the `github.token` binding, and removes the direct
secret reveal it replaces. Delivers `engineering:core-provider-split`, and `engineering:shared-auth-boundary` and
`engineering:accounting-by-authorization` for supplied tokens. Builds [authorization.md](../spec/authorization.md) for
`github.token`, the `token-unsupported` row of [refusals.md](../spec/refusals.md#interfaces), and the hub-start and
pass-start rows of the gates in [readiness-check.md](../spec/readiness-check.md#interfaces). Shaped by
`decision:advanced-forge-auth/fine-grained-checks`.

## Change

- `IForgeAuthorizations` and `resolve_credential`; a referrer with no `credential_id` resolves to an implicit
  `github.token`, so the slice lands before the carry-over.
- `ForgeRegistry` and `WorkSourceEntry.built` stop revealing secrets; both clients authenticate through `httpx.Auth`.
- `BudgetKey` and `WriteLocks` key on `(forge_id, authority)`; `CacheKey` carries the credential's revision.
- The effective variant narrows `ForgeRegistry.offered(forge, repository)`; the pass-start `forge-capability-missing`
  event names the credential; hub start warns about an undeclared fine-grained token behind a policy it cannot serve.
- Leaves alone: what a repository with no credential sends, and its budget, which keys on `secret/<secret_name>` as it
  keyed on the secret's name.

## Acceptance

- The layering guard permits `ISecretReader.reveal` only under `hub/forge/internal/`.
- Effective-variant classification passes for every row of its table.
- With `checks.read` withheld, a repository whose token is fine-grained behind `merge-on-checks` escalates
  `forge-capability-missing` at a pass's start, naming the credential, with no forge write; hub start logs the warning
  for it and none for a classic token.
- A 403 on the check read for a fine-grained token is classified `token-unsupported`.
- Replacing a credential's secret changes the token the next request sends, with no restart.
- A repository and a work source naming one token share one budget and one write lane.
