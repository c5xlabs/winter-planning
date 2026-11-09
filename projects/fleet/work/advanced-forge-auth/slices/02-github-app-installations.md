---
name: github-app-installations
depends-on:
  - explicit-credentials
---

# GitHub App installations

Opens with the private-repository Checks proof, then delivers App setup, acquisition, and renewal across every hub
caller. Delivers `product:github-app-installations`, `engineering:private-checks-proof`,
`engineering:managed-token-renewal`, `engineering:accounting-by-authorization`, and `engineering:shared-auth-boundary`
for managed access.

## Change

- The proof runs first, as a gate: a test App's installation token reads a private repository's pull request head and
  the exact check-runs endpoint delivery uses.
- Administrators register an App identity, installation, and private-key secret in their tenant.
- The hub obtains, narrows, and renews installation tokens for every hub caller, including expiry, concurrent refresh,
  revocation, and key replacement.
- Setup and recovery instructions ship with the implementation.

## Acceptance

- The proof verifies the expected checks and conclusions, opens no pull request, and performs no merge.
- Tests against the mock forge cover expiry, concurrent refresh, restart, revocation, suspension, removed repositories,
  changed permissions, and key replacement.
- No cached token is served across tenants, forge servers, installations, or access scopes.
