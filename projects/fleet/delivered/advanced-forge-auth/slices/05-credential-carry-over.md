---
name: credential-carry-over
external: github:paul-gross/blizzard#2178
depends-on:
  - credential-record
code:
  - blizzard/src/blizzard/hub/store/schema.py::repositories
  - blizzard/src/blizzard/hub/store/schema.py::work_sources
---

# Credential carry-over

Gives every existing repository and work source a credential, linking any a door already created. Delivers
`engineering:additive-transition` for stored data. Builds the `<ts>_credential_carry_over` revision of
credential-record.md's [Data](../spec/credential-record.md#data). Shaped by
`decision:advanced-forge-auth/credential-carry-over`.

## Change

- The revision links each referrer to the credential holding its authority, creates the rest, and links again.
- Leaves alone: `secret_name`, which stays mirrored, and credentials a door created.

## Acceptance

- The revision's test: a shared secret yields one credential for both uses; a secret on two forges yields two with slugs
  distinct from every current and former credential slug; a work source with no secret keeps none; a retired secret
  yields a retired credential.
- On a store where the API already created a credential for an authority, the revision links its referrers to it and
  creates none.
- The downgrade clears only the links to credentials it created, and deletes only those.
- Every invariant holds after the revision on a store seeded with each case.

## Out of scope

- Retiring `secret_name`: a separate cleanup after the epic, per `engineering:additive-transition`.
