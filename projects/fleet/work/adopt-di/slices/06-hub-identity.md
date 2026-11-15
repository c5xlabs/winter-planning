---
name: hub-identity
depends-on:
  - hub-request-pilot
code:
  - blizzard/src/blizzard/hub/composition.py::build_services
  - blizzard/src/blizzard/hub/api/auth_session.py::resolve_identity
  - blizzard/src/blizzard/hub/api/auth_session.py::resolve_tenant
---

# Hub identity wiring and auth dependencies

Delivers the hub-identity row in spec [Mixed ownership](../spec/migration.md#area-ownership), with process-backed auth
resolution from spec [Entrypoint resolution](../spec/entrypoints.md). Serves `engineering:area-wiring`,
`engineering:narrow-api-edge`, and `engineering:scope-identity`; follows `decision:adopt-di/declared-cutover`.

## Change

- Move identity/session/membership/admin/signing/auth-state/facts/throttle providers and tenant invitation construction
  into auth wiring, retaining their process-versus-tenant lifetimes.
- Narrow auth, login, identity-provider, self, member and hub-admin routes plus session/machine-credential dependencies.
- Remove hub-service references from auth request records/helpers; retain only explicit principal and scoped values.

## Acceptance

- Auth/admission component tests preserve cookie/bearer precedence, machine identity, membership and special route
  admission.
- Hub-only routes resolve without a tenant; request identities never appear in process or tenant providers.
- No owned auth provider is duplicated and no migrated auth helper accepts a service bag.
- Strict typing, graph/route gates and existing auth/API tests pass; available fields/bridges are removed.
