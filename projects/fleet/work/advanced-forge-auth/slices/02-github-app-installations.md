---
name: github-app-installations
depends-on:
  - explicit-credentials
code:
  - blizzard/src/blizzard/hub/forge/internal/factory.py::build_forge_registry
  - blizzard/src/blizzard/hub/forge/budget.py::ForgeBudgets
  - blizzard/web/projects/hub/src/app/admin/config-edit.model.ts::ConfigFieldKind
---

# GitHub App installations

Adds the `github.app-installation` binding: the private-repository proof first, then the mock forge's Apps, then minting
and renewal across every hub caller. Delivers `product:github-app-installations`, `engineering:private-checks-proof`,
`engineering:managed-token-renewal`, and `engineering:shared-auth-boundary` and
`engineering:accounting-by-authorization` for managed access. Builds the App parts of
[authorization.md](../spec/authorization.md), including its proof tool and [Flows](../spec/authorization.md#flows); the
mint rows of [refusals.md](../spec/refusals.md#interfaces); the App parts of [mock-forge.md](../spec/mock-forge.md); and
the board's `'secret-text'` field from [credential-record.md](../spec/credential-record.md#interfaces). Shaped by
`decision:advanced-forge-auth/tokens-in-memory` and `decision:advanced-forge-auth/no-app-registration-record`; settles
`decision:advanced-forge-auth/fine-grained-checks` and the open question in
[readiness-check.md](../spec/readiness-check.md#open-questions).

## Change

- The proof tool runs against an operator-approved private repository before any App code merges; its result rewrites
  `decision:advanced-forge-auth/fine-grained-checks`, and `FINE_GRAINED_WITHHELD` and the mock forge's fine-grained row
  follow it.
- `blizzard-mock`'s forge serves App installations, JWT-verified mints, and the App levers.
- `InstallationTokens` mints, narrows to the scope, renews, and invalidates installation tokens for landing, the
  garden's commit check, and work sources; the runtime offer narrows to the installation's granted permissions.
- The board's secret form takes a multi-line PEM key.
- Docs: `docs/deployment/credentials.md` gains App creation, installation, the permissions each scope needs, key
  replacement, and recovery from each App refusal.

## Acceptance

- The proof's App line reads `pull=200 check-runs=200 matched=yes`, and the decision records each credential's line.
- On the controlled clock: a token within five minutes of `expires_at` is renewed on the next request; concurrent
  requests from threads and the action loop cause one mint; a restart mints on first use; a replaced key mints under the
  new revision; a 401 drops the token and retries once.
- No two tenants, forges, installations, or scopes ever receive the same cached token.
- On the mock forge with a short `TOKEN_LIFETIME`, one installation lands several repositories and sweeps a work source
  across token expiry and a hub restart with no operator action, under one budget and one write lane.
- A revoked token, a suspended installation, a deleted installation, a removed repository, a withdrawn permission, and a
  replaced key each produce their own refusal and remedy.
- An installation without `checks: read` behind `merge-on-checks` escalates `forge-capability-missing` at a pass's
  start.
