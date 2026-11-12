---
name: credential-document-and-board
external: github:paul-gross/blizzard#2179
depends-on:
  - credential-api-and-cli
  - credential-carry-over
code:
  - blizzard/src/blizzard/wire/config.py::ConfigDocumentV2
  - blizzard/src/blizzard/hub/domain/config/apply.py::reconcile
  - blizzard/src/blizzard/hub/api/config.py::DOCUMENT_VERSION
  - blizzard/web/projects/hub/src/app/admin/admin.routes.ts::ADMIN_ROUTES
  - blizzard/web/projects/hub/src/app/admin/config-links.model.ts::SURFACE_SEGMENT
  - blizzard/web/projects/hub/src/app/admin/repositories/repository-detail.ts::RepositoryDetail
---

# Credential document and board

Adds credentials to the configuration document as version 3 and to the board. Delivers the document and board parts of
`product:credential-records`. Builds the configuration document and the board of credential-record.md's
[Interfaces](../spec/credential-record.md#interfaces), apart from the App binding's fields and the `'secret-text'`
field.

## Change

- `ConfigDocumentV3`, `CredentialEntry`, and the v3 repository and work-source entries; apply reconciles `credentials`
  through `PlannedCredentials` and resolves v2 old fields through `credential_for_old_field`; export and schema take
  `version`, defaulting to 2.
- `admin/credentials/` with the list, form, and detail; repository and work-source forms pick a credential filtered to
  their forge; the board's record-kind enumerations learn the credential.

## Acceptance

- A v2 document applies unchanged and exports byte-identical to before the slice.
- A v3 document declaring a credential and its referrers applies in one apply and exports back to itself.
- A v3 entry naming a credential of another forge is refused.
- Board component tests create, edit, retire, and enable a credential, show its revisions and referrers, and offer a
  repository only credentials of its forge.
