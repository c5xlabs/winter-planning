---
name: hub-identity
external: github:paul-gross/blizzard#2370
depends-on:
  - hub-store-adapters
  - hub-configured-adapters
code:
  - blizzard/src/blizzard/hub/composition.py::build_services
  - blizzard/src/blizzard/hub/api/auth_session.py::resolve_identity
  - blizzard/src/blizzard/hub/api/auth_session.py::resolve_tenant
  - blizzard/src/blizzard/hub/api/fleet_identity.py::get_runner_identity
---

# Hub identity wiring and auth dependencies

Delivers the hub-identity row in spec [Mixed ownership](../spec/migration.md#area-ownership), with process-backed auth
resolution from spec [Entrypoint resolution](../spec/entrypoints.md). Serves `engineering:area-wiring`,
`engineering:narrow-api-edge`, and `engineering:scope-identity`; follows `decision:adopt-di/declared-cutover`.

## Change

- Move identity/session/membership/admin/signing/auth-state/facts/throttle providers, `IdentityTransactions` (replacing
  the legacy leaf provider the store-adapter change registered), tenant invitation construction over the registered
  `InvitationStore`, the `hub/auth/internal` repositories and `UserRepository`, `TrustedProxies`, and the owned OAuth
  HTTP client and its cleanup into auth wiring, retaining their process-versus-tenant lifetimes. Their configured inputs
  (OAuth providers, the signing-keys directory, trusted proxies, the login throttle) come from the `HubConfig` source
  the configured-adapter change registered; the OAuth client takes the platform-tracing `instrument_client` hook.
- Narrow auth, login, identity-provider, self, member and hub-admin routes plus session/machine-credential dependencies;
  `reject_runner_principal`, `require` and `granted` resolve their named process services from the resolver
  `process_request` attached, completing the sequence the request pilot started for `resolve_tenant`.
- Narrow the runner identity route (`fleet_identity.get_runner_identity`) to its machine-credential resolution and the
  `OpenTenants` read (a legacy leaf provider until hub lifecycle adopts it).
- Remove hub-service references from auth request records/helpers; retain only explicit principal and scoped values.

## Acceptance

- Auth/admission component tests preserve cookie/bearer precedence, machine identity, membership and special route
  admission.
- Hub-only routes resolve without a tenant; request identities never appear in process or tenant providers.
- No owned auth provider is duplicated and no migrated auth helper accepts a service bag.
- `blizzard:component-test`, `blizzard:service-test`, `blizzard:typecheck` and the graph/route gates pass; available
  fields/bridges are removed.
