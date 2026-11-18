---
name: app-installation-binding
external: github:paul-gross/blizzard#2182
depends-on:
  - authorization-boundary
  - credential-document-and-board
  - mock-forge
code:
  - blizzard/src/blizzard/hub/forge/internal/factory.py::build_forge_registry
  - blizzard/src/blizzard/hub/forge/budget.py::ForgeBudgets
  - blizzard/web/projects/hub/src/app/admin/config-edit.model.ts::ConfigFieldKind
---

# App installation binding

Adds the `github.app-installation` binding at every door and behind the boundary, with its docs and its end-to-end
proof. Delivers `product:github-app-installations`, `engineering:managed-token-renewal`, and
`engineering:shared-auth-boundary` and `engineering:accounting-by-authorization` for managed access. Builds the App
parts of [authorization.md](../spec/authorization.md), its [Flows](../spec/authorization.md#flows), the mint rows of
[refusals.md](../spec/refusals.md#interfaces), the `<ts>_app_credentials` revision of credential-record.md's
[Data](../spec/credential-record.md#data), and the App settings and `'secret-text'` field of its
[Interfaces](../spec/credential-record.md#interfaces). Shaped by `decision:advanced-forge-auth/tokens-in-memory`;
follows `engineering:private-checks-proof`, which already holds.

## Change

- `CREDENTIAL_BINDINGS` gains `github.app-installation` with its revision, so no store or door accepts an App credential
  before the runtime can use one.
- `InstallationTokens` mints, narrows, renews, and invalidates tokens per tenant, forge, authority, scope, and key
  revision; the runtime offer narrows to the installation's granted permissions.
- `--app-id` and `--installation-id` on `blizzard hub credential`; the board's App fields and multi-line masked secret
  field.
- `docs/deployment/credentials.md` gains App creation and installation, each scope's permissions, registering the
  credential, key replacement, and recovery from each App refusal.

## Acceptance

- On the controlled clock: a token within five minutes of `expires_at` is renewed on the next request; concurrent
  requests from threads and the action loop cause one mint; a restart mints on first use; a replaced key mints under its
  new revision; a 401 drops the token and retries once.
- No two tenants, forges, installations, or scopes receive the same cached token, and the `<ts>_app_credentials`
  downgrade is refused while an App credential exists.
- On the mock forge with a short `TOKEN_LIFETIME`, one installation lands several repositories and sweeps a work source
  across token expiry and a hub restart with no operator action, under one budget and one write lane.
- An expired token the hub cannot renew, a revoked token, a suspended installation, a deleted installation, a removed
  repository, a withdrawn permission, and a replaced key each produce their own refusal and remedy.
- An installation without `checks: read` behind `merge-on-checks` escalates `forge-capability-missing` at a pass's
  start.
- A PEM key pasted into the board's secret form reaches the API with its line breaks, and the permissions
  `credentials.md` lists equal the scope table's.
