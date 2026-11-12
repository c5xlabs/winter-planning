---
name: supplied-token-proof-and-docs
depends-on:
  - authorization-boundary
  - credential-api-and-cli
  - credential-carry-over
  - mock-forge
---

# Supplied-token proof and docs

Proves the supplied-token path end to end on the mock forge and documents it for operators. Delivers the end-to-end and
documentation acceptance of `product:supplied-tokens` and `product:credential-records`. Builds `FINE_GRAINED_WITHHELD`
per `decision:advanced-forge-auth/fine-grained-checks`.

## Change

- Fleet-tier tests over a hub, runner, and the mock forge under `STRICT_TOKENS`.
- `docs/deployment/credentials.md`: the record, PAT setup for classic and fine-grained tokens, the permissions each
  supports, and what is refused; `repositories.md`, `work-sources.md`, `forges.md`, and `config-documents.md` point to
  it.

## Acceptance

- After the carry-over, an existing repository and work source keep ingesting and landing with no token re-entered.
- A classic token lands a private repository through `merge-on-checks`.
- A carried-over credential whose token is fine-grained escalates `forge-capability-missing` at a pass's start, before
  any forge write, while the decision withholds `checks.read`.
- A revoked token escalates `forge-auth` naming the credential and records `forge-auth-refused`.
- Every command and field `credentials.md` names exists in `contracts/cli/hub.json` or `openapi/hub.openapi.json`, and
  the docs' link checks pass.
- Pointing a repository and a work source at another credential makes their next requests send its token, with no
  restart.
