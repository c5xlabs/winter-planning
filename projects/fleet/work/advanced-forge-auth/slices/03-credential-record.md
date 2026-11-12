---
name: credential-record
depends-on:
  - epic:advanced-delivery/forges
code:
  - blizzard/src/blizzard/hub/store/schema.py::repositories
  - blizzard/src/blizzard/hub/store/schema.py::work_sources
  - blizzard/src/blizzard/hub/domain/config/authoring.py::ConfigAuthoring
  - blizzard/src/blizzard/hub/domain/config/changes.py::RecordKind
  - blizzard/src/blizzard/hub/store/internal/invariants.py::HubQueryCheck
  - blizzard/src/blizzard/hub/store/internal/secret_store.py::lock_secret_row
---

# Credential record

Adds the credential as a configured record, with nothing calling it yet but `ConfigAuthoring`. Delivers the record part
of `product:credential-records` and `engineering:no-credential-crosses-tenants`. Builds the
[Domain](../spec/credential-record.md#domain) of credential-record.md and the `<ts>_credentials` revision of its
[Data](../spec/credential-record.md#data). Shaped by `decision:advanced-forge-auth/one-credential-per-authority` and
`decision:advanced-forge-auth/no-app-registration-record`. Builds `FINE_GRAINED_WITHHELD` as
`decision:advanced-forge-auth/fine-grained-checks` stands when it lands.

## Change

- The `credentials`, `credential_former_slugs`, and `credential_lifecycle_facts` tables, and nullable `credential_id` on
  repositories and work sources.
- `CredentialBinding`, `CREDENTIAL_BINDINGS` holding `github.token` only, `FINE_GRAINED_WITHHELD`,
  `ConfiguredCredential`, and its read and write repositories.
- `ConfigAuthoring`'s four credential verbs, `lock_credential_row`, and the referrer checks on retiring a secret or a
  forge.
- The six invariants, the change-log enumerations on the hub side, the store-read census recipes, and
  `contracts/store/hub.json`.

## Acceptance

- The revision's test creates and downgrades the schema on SQLite and Postgres.
- `ConfigAuthoring` creates, edits, retires, and enables a credential, records each change, and refuses each of
  `credential_binding_unknown`, `credential_forge_kind_mismatch`, `credential_settings_invalid`,
  `credential_binding_immutable`, `credential_authority_taken`, and `credential_in_use`.
- Retiring a secret or a forge an enabled credential names is refused.
- Each invariant fails on a store seeded to break it and passes on one that does not.
- One tenant's credentials are invisible to another tenant's reads and writes.
