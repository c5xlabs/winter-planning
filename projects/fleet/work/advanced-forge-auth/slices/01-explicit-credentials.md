---
name: explicit-credentials
depends-on:
  - epic:advanced-delivery/forges
---

# Explicit credentials

Establishes the provider-aware credential boundary and tenant-owned credential records, wires repositories and work
sources through it, and carries existing PAT setup forward. Delivers `product:credential-records`,
`product:supplied-tokens`, `engineering:core-provider-split`, `engineering:shared-auth-boundary` for supplied tokens,
`engineering:no-credential-crosses-tenants`, and `engineering:additive-transition`.

## Change

- Tenants manage typed credential records through the API, CLI, configuration document, and board.
- Repositories and forge-backed work sources reference a credential through the boundary.
- Existing secret references keep working; GitHub setup tells classic and fine-grained PATs apart and refuses known
  unsupported combinations.
- The port preserves actionable provider refusals.
- Ships its tests, operator documentation, and compatible upgrade path.

## Acceptance

- The fleet ingests and lands through supported PATs at the end of the slice.
- An existing setup upgrades without re-entering a token.
- No surface or diagnostic shows a token or key.
