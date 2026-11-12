---
name: explicit-credentials
depends-on:
  - epic:advanced-delivery/forges
code:
  - blizzard/src/blizzard/hub/store/schema.py::repositories
  - blizzard/src/blizzard/hub/store/schema.py::work_sources
  - blizzard/src/blizzard/hub/domain/config/authoring.py::ConfigAuthoring
  - blizzard/src/blizzard/hub/forge/registry.py::ForgeRegistry
  - blizzard/src/blizzard/hub/forge/registry.py::ForgeBinding
  - blizzard/src/blizzard/hub/forge/internal/github_landing.py::GitHubLanding
  - blizzard/src/blizzard/hub/forge/budget.py::BudgetKey
  - blizzard/src/blizzard/hub/live_config/__init__.py::CacheKey
  - blizzard/src/blizzard/hub/work_sources/internal/factory.py::WorkSourceEntry
  - blizzard/src/blizzard/hub/work_sources/internal/github_work_source.py::GitHubWorkSource
  - blizzard/src/blizzard/hub/delivery/actions/landing/repo.py::RepoLanding
---

# Explicit credentials

Adds the credential record, carries every existing secret reference onto it, and moves landing and ingestion onto the
authorization boundary with the `github.token` binding and classified refusals. Delivers `product:credential-records`,
`product:supplied-tokens`, and the refusal half of `product:actionable-refusals`; `engineering:core-provider-split`,
`engineering:no-credential-crosses-tenants`, `engineering:secrets-out-of-evidence`, `engineering:additive-transition`,
and `engineering:shared-auth-boundary` and `engineering:accounting-by-authorization` for supplied tokens. Builds
[credential-record.md](../spec/credential-record.md), the token parts of [authorization.md](../spec/authorization.md),
[refusals.md](../spec/refusals.md) but its mint rows and recovery remedy, the configuration-write and pass-start gates
of [readiness-check.md](../spec/readiness-check.md#interfaces), and the token levers of
[mock-forge.md](../spec/mock-forge.md). Shaped by `decision:advanced-forge-auth/credential-carry-over`,
`decision:advanced-forge-auth/one-credential-per-authority`, and `decision:advanced-forge-auth/fine-grained-checks`.

## Change

- Tenants create, edit, retire, and enable credentials through the API, CLI, configuration document version 3, and the
  board's Credentials tab; repositories and work sources name one.
- The migration gives every existing repository and work source a credential; `secret_name` and `secret` stay accepted
  and mirrored.
- `ForgeAuthorizations` is the one revealer of forge secrets; landing and ingestion clients authenticate through
  `httpx.Auth`; budgets, write locks, and the adapter cache key on the credential.
- A fine-grained token is offered less than the forge kind, so the existing capability checks refuse a landing policy it
  cannot serve.
- 401 and 403 answers become `ForgeAuthRefused` with a sanitized diagnostic, in the `forge-auth` escalation detail and a
  `forge-auth-refused` event, for landing and work sources.
- `blizzard-mock`'s forge gains `STRICT_TOKENS`, `REGISTER_TOKEN`, `REVOKE_TOKEN`, `PRIVATE_REPOSITORY`, and request
  ids.
- Docs: `docs/deployment/credentials.md` is added; `repositories.md`, `work-sources.md`, `forges.md`,
  `config-documents.md`, and `observability.md` name the credential, and `repositories.md` drops its claim that a
  fine-grained token reads checks. `blizzard-context`'s `domain/operations.md`, `domain/projects.md`,
  `domain/humans/escalation.md`, and `architecture/system-shape/configuration.md` and `seam-positions.md` learn the
  credential, the boundary, and the event.
- Leaves alone: the runner's checkout credentials, and every escalation cause and its routing.

## Acceptance

- The migration test: a shared secret yields one credential for both uses; a secret on two forges yields two; a work
  source with no secret keeps none; a retired secret yields a retired credential; the downgrade restores the schema.
- After upgrade on the mock forge, an existing repository and work source keep ingesting and landing with no token
  re-entered.
- Each refusal in the credential and referrer tables is returned for its case at the API, and the CLI and board create,
  edit, retire, and enable a credential and show its revisions.
- A v2 document applies unchanged; a v3 document declaring a credential and its referrers applies in one apply.
- Under `STRICT_TOKENS`, a classic token lands a private repository through `merge-on-checks`.
- A fine-grained credential behind `merge-on-checks` is refused `422 forge_capability_missing` at configuration; a
  carried-over credential whose token is fine-grained escalates `forge-capability-missing` at a pass's start with no
  forge write.
- A revoked token escalates `forge-auth` with the operation, status, GitHub message, request id, and credential in its
  detail, and records `forge-auth-refused`; a work source's sweep under a revoked token records the event naming it.
- Every row of the classifier table maps its answer; the sanitizer's property test finds no token, JWT, PEM block, or
  `Authorization` value in its output; no response, log, event, or escalation in the suite carries the test token.
- A refused, unreadable, or empty check list never merges.
- Replacing a credential's secret changes the token the next request sends, with no restart.

## Out of scope

- Retiring `secret_name`, `secret`, and the v2 document: a separate cleanup after the epic, per
  `engineering:additive-transition`.
