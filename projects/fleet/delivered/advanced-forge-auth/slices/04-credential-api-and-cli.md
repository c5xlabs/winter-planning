---
name: credential-api-and-cli
external: github:paul-gross/blizzard#2177
depends-on:
  - credential-record
code:
  - blizzard/src/blizzard/hub/api/repositories.py::create_repository
  - blizzard/src/blizzard/hub/api/work_sources.py::create_work_source
  - blizzard/src/blizzard/wire/repository.py::RepositoryCreateRequest
  - blizzard/src/blizzard/wire/work_source.py::WorkSourceCreateRequest
  - blizzard/src/blizzard/hub/domain/config/landing_policies.py::require_offered
  - blizzard/src/blizzard/hub/cli/__init__.py::_COMMANDS
  - blizzard/src/blizzard/hub/cli/repo.py::repo_create
  - blizzard/src/blizzard/hub/cli/source.py::source_create
  - blizzard/src/blizzard/hub/cli/config.py::_KINDS
---

# Credential API and CLI

Opens `github.token` credentials to the API and CLI, lets repositories and work sources name one, and keeps the old
secret fields working. Delivers the API and CLI parts of `product:credential-records` and `product:supplied-tokens`, and
`engineering:additive-transition` for both doors. Builds the API and CLI of credential-record.md's
[Interfaces](../spec/credential-record.md#interfaces), apart from the App binding's settings, and the
configuration-write and credential-edit gates of [readiness-check.md](../spec/readiness-check.md#interfaces). Builds
`FINE_GRAINED_WITHHELD` per `decision:advanced-forge-auth/fine-grained-checks`.

## Change

- `/api/credentials` and its six routes, `PathCredential`, and `wire/credential.py`; `openapi/hub.openapi.json` and the
  board's client are regenerated.
- Repository and work-source requests take `credential`; their summaries carry the credential and `warnings`.
- `credential_for_old_field` resolves an old secret field, creating a credential when needed, with a `Deprecation`
  header, a warning, and `config-old-field-used`.
- `require_offered` reads `declared_offers`; editing a credential re-checks its enabled referrers.
- `blizzard hub credential` with create, list, show, edit, retire, and enable for `github.token`; `--credential` on
  `repo` and `source`; `credential` among `config changes --kind`; `contracts/cli/hub.json` regenerated.

## Acceptance

- Each refusal in the API's two refusal tables is returned for its case.
- Every row of the old-field table produces its result, and two concurrent writes resolving one new authority create one
  credential.
- A client body written before this slice still validates, and its response still parses.
- A fine-grained credential behind `merge-on-checks` is refused `422 forge_capability_missing` at create, and an edit
  making a credential fine-grained behind one is refused, listing the referrer.
- Each CLI command round-trips a credential against a service-tier hub, `blizzard hub config changes --kind credential`
  lists its history, and `--secret` alone prints the old-field warning.
