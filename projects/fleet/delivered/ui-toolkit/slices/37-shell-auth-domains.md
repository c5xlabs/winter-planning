---
name: shell-auth-domains
external: github:paul-gross/blizzard#2291
depends-on:
  - shared-core-domains
code:
  - blizzard/web/projects/hub/src/app/shell/app.ts::App
  - blizzard/web/projects/hub/src/app/core/auth/me.query.ts::injectMeQuery
---

# Move shell and authentication readers onto domains

Completes [boundary.md](../spec/boundary.md#system)'s shell/auth/lens read boundary in both apps for
`product:domain-boundary`, preserving `decision:ui-toolkit/nav-stays-links`.

## Change

- Translate session, provider, tenant/project lens, header, and filter data into the appropriate shared or app domain.
- Update permission and sign-in consumers without changing authorization, invitation landing, or return-url behavior.
- Preserve routed main-nav links and the existing eager-shell dependency boundary.

## Acceptance

- Tests cover signed-out, memberless, multi-tenant, refused-sign-in, and selected-lens states with domain fixtures.
- Existing return-url, invitation, permission, route, and shell tests remain green.
- `web:typecheck`, `web:unit-test`, and `web:bundle-composition` pass; moved shell/auth units leave exemptions.
