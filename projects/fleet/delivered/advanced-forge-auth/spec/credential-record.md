# Credential record

Paths are relative to `blizzard/src/blizzard/`.

## Domain

```python
# hub/domain/config/credential_bindings.py — beside FORGE_KIND_CAPABILITIES; imports no adapter
class CredentialBinding(Protocol):
    name: str                       # "<forge kind>.<binding>", e.g. "github.token"
    forge_kind: str                 # must equal the credential's forge's kind
    supplied: bool                  # True: the operator replaces the secret. False: the hub renews access.
    def validate(self, settings: Mapping[str, object]) -> Mapping[str, object]: ...
        # canonical settings, or raises CredentialSettingsInvalid(field)
    def authority(self, secret_name: str, settings: Mapping[str, object]) -> str: ...
    def declared_offers(self, settings: Mapping[str, object], kind: frozenset[Capability]) -> frozenset[Capability]: ...
        # what configuration alone says the credential can do; the runtime may narrow it further

CREDENTIAL_BINDINGS: Mapping[str, CredentialBinding]   # "github.token", "github.app-installation"
FINE_GRAINED_WITHHELD: frozenset[Capability] = frozenset({Capability.CHECKS_READ})
    # decision:advanced-forge-auth/fine-grained-checks
```

| Binding                   | `settings`                                                    | `authority`                                   | `declared_offers`                                          |
| ------------------------- | ------------------------------------------------------------- | --------------------------------------------- | ---------------------------------------------------------- |
| `github.token`            | `variant`: `"classic"`, `"fine-grained"`, or absent           | `secret/<secret_name>`                        | kind set; less `FINE_GRAINED_WITHHELD` when `fine-grained` |
| `github.app-installation` | `app_id`, `installation_id`: positive integers, both required | `app/<app_id>/installation/<installation_id>` | kind set                                                   |

```python
# hub/domain/config/credentials.py
@dataclass(frozen=True)
class CredentialFields:
    slug: str
    name: str
    forge_id: ForgeId          # immutable after create
    binding: str               # immutable after create
    secret_name: str
    settings: Mapping[str, object]

class ConfiguredCredential:    # SluggedRetirableConfiguredRecord, patterned on ConfiguredForge
    credential_id: CredentialId          # IdPrefix.CREDENTIAL = "cred"
    fields: CredentialFields
    authority: str                       # derived; never written by a door
    revision: int
    retired: bool
    @classmethod
    def new(cls, fields, *, binding: CredentialBinding, at, by) -> tuple["ConfiguredCredential", ConfigChange]: ...
    def edit(self, edit: CredentialEdit) -> tuple["ConfiguredCredential", ConfigChange] | None: ...
    def set_retired(self, retired: bool) -> tuple["ConfiguredCredential", ConfigChange] | None: ...

class IReadCredentialRepository(Protocol): ...
class IWriteCredentialRepository(Protocol): ...   # joins the configured-write gate in tests/guards/test_layering.py
```

`ConfigAuthoring` (`hub/domain/config/authoring.py`) gains `create_credential`, `edit_credential`, `retire_credential`,
and `enable_credential`. `RecordKind.CREDENTIAL`, keyed by id, joins `hub/domain/config/changes.py`; repository and
work-source changes gain a `credential` field labelled by slug. Every enumeration a new configured kind joins learns it,
as listed in [the forges record spec](../../../delivered/advanced-delivery/forges/spec/record.md#the-change-log).

Invariants, in `hub/store/internal/invariants.py`:

| Id                                    | Holds                                                                                     |
| ------------------------------------- | ----------------------------------------------------------------------------------------- |
| `hub:credential-references-in-tenant` | A referrer's `credential_id` names a credential of its tenant                             |
| `hub:credential-forge-matches`        | A referrer's credential names the referrer's forge                                        |
| `hub:credential-secret-mirrored`      | A referrer's `secret_name` equals its credential's `secret_name`                          |
| `hub:credential-reference-standing`   | An enabled referrer names an enabled credential, whose forge is enabled and secret active |
| `hub:credential-authority-derived`    | `authority` equals what the binding derives from the stored settings                      |
| `hub:slug-not-current-and-former`     | Gains `credential` among its kinds                                                        |

## Data

```python
# hub/store/schema.py — TENANT_TABLES
credentials = Table(
    "credentials", metadata,
    Column("credential_id", String, nullable=False),     # hub_unique
    Column("tenant_id", String, nullable=False),
    Column("slug", String, nullable=False),
    Column("name", String, nullable=False),
    Column("forge_id", String, nullable=False),
    Column("binding", String, nullable=False),
    Column("secret_name", String, nullable=False),       # the token, or the App's PEM private key
    Column("settings", JSON, nullable=False),
    Column("authority", String, nullable=False),
    Column("revision", Integer, nullable=False),
    Column("created_at", DateTime(timezone=True), nullable=False),
    Column("created_by", String, nullable=False),
    PrimaryKeyConstraint("tenant_id", "credential_id"),
    UniqueConstraint("tenant_id", "slug"),
    UniqueConstraint("tenant_id", "forge_id", "authority"),   # decision:advanced-forge-auth/one-credential-per-authority
    ForeignKeyConstraint(["tenant_id", "forge_id"], ["forges.tenant_id", "forges.forge_id"]),
    ForeignKeyConstraint(["tenant_id", "secret_name"], ["secrets.tenant_id", "secrets.name"]),
)
# credential_former_slugs, credential_lifecycle_facts: the shape of forge_former_slugs, forge_lifecycle_facts

# repositories, work_sources — engineering:additive-transition, bzh:store-expand-contract
Column("credential_id", String, nullable=True)            # no FK until the old field retires
Column("secret_name", ..., info={"deprecated": "<line>"}) # kept; every write sets it to the credential's secret_name
```

Three additive Alembic revisions in `hub/store/migrations/versions/`, each with a colocated test:

| Revision                     | Does                                                                                         | Downgrade                                                                                                   |
| ---------------------------- | -------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `<ts>_credentials`           | Creates the three tables and the two nullable `credential_id` columns                        | Drops them                                                                                                  |
| `<ts>_credential_carry_over` | Links or creates a credential for every referrer, as below                                   | Removes what it created, as below                                                                           |
| `<ts>_app_credentials`       | No schema change; marks the store as one that may hold `github.app-installation` credentials | Refused while one exists, so no build that sends `secret_name` as a token serves a store holding an App key |

The carry-over:

```sql
-- per tenant; decision:advanced-forge-auth/credential-carry-over
-- 1. link: a referrer with no credential_id takes the credential already holding its authority, if any
UPDATE repositories r SET credential_id = (SELECT c.credential_id FROM credentials c
  WHERE (c.tenant_id, c.forge_id, c.authority) = (r.tenant_id, r.forge_id, 'secret/' || r.secret_name))
WHERE r.credential_id IS NULL;
-- the same for work_sources with a secret_name

-- 2. create: one github.token credential per (forge_id, secret_name) a referrer still names with no credential_id
INSERT INTO credentials (credential_id, tenant_id, slug, name, forge_id, binding, secret_name, settings, authority,
                         revision, created_at, created_by)
SELECT new_cred_id(), tenant_id, unique_slug(tenant_id, secret_name), secret_name, forge_id, 'github.token',
       secret_name, '{}', 'secret/' || secret_name, 1, now(), 'migration:credential-carry-over'
FROM (SELECT tenant_id, forge_id, secret_name FROM repositories WHERE credential_id IS NULL
      UNION SELECT tenant_id, forge_id, secret_name FROM work_sources
            WHERE credential_id IS NULL AND secret_name IS NOT NULL);
-- unique_slug: the secret's name, else "-2", "-3", ...; never a current or former credential slug of the tenant
-- a credential whose secret is retired gets a retired lifecycle fact
-- each creation writes a config_changes row as the forges migration wrote its forges'

-- 3. link again, as in 1; afterwards only a work source with no secret has credential_id NULL
-- 4. enable: a retired credential an enabled referrer now names is enabled, with a lifecycle fact by
--    migration:credential-carry-over; the downgrade does not retire it again
```

The downgrade sets `credential_id` to NULL on every referrer naming a credential whose `created_by` is
`migration:credential-carry-over`, then deletes those credentials, their lifecycle facts, and their change rows.
Credentials a door created are left in place.

`contracts/store/hub.json` is regenerated. `tests/support/store_read_census.py` gains a recipe for each new `IRead*`
method.

## Interfaces

```text
GET   /api/credentials                          FLEET_VIEW    -> CredentialsListView
POST  /api/credentials                          CONFIG_EDIT   CredentialCreateRequest -> 201 CredentialSummary
GET   /api/credentials/{credential}             FLEET_VIEW    -> CredentialDocument (with referrers)
PATCH /api/credentials/{credential}             CONFIG_EDIT   If-Match; CredentialPatchRequest -> CredentialSummary
POST  /api/credentials/{credential}/retire      CONFIG_EDIT   If-Match
POST  /api/credentials/{credential}/enable      CONFIG_EDIT   If-Match
```

`{credential}` is an id, slug, or former slug, resolved by `PathCredential` in `hub/api/credential_params.py`.

```python
# wire/credential.py
class CredentialCreateRequest(BaseModel):  # extra="forbid"
    slug: Slug; name: str | None; forge: IdOrSlug; binding: str; secret: SecretNameStr; settings: dict[str, Any] = {}
class CredentialPatchRequest(BaseModel):   # extra="forbid"; any subset
    slug: Slug; name: str; secret: SecretNameStr; settings: dict[str, Any]
class CredentialSummary(BaseModel):
    credential_id: str; slug: str; name: str; forge_id: str; forge_slug: str; forge_name: str
    binding: str; secret_name: str; settings: dict[str, Any]; authority: str; revision: int; retired: bool
```

| Refusal                                   | Status | When                                                                             |
| ----------------------------------------- | ------ | -------------------------------------------------------------------------------- |
| `credential_binding_unknown`              | 422    | `binding` is not in `CREDENTIAL_BINDINGS`                                        |
| `credential_forge_kind_mismatch`          | 422    | The binding's `forge_kind` is not the forge's kind                               |
| `credential_settings_invalid`             | 422    | `validate` refused; names the field                                              |
| `credential_binding_immutable`            | 422    | A patch carries `forge` or `binding`                                             |
| `credential_authority_taken`              | 409    | Another credential on the forge has the authority; names it                      |
| `credential_in_use`                       | 409    | Retire while an enabled repository or work source names it; lists the referrers  |
| `forge_capability_missing`                | 422    | A patch narrows `declared_offers` below an enabled referrer's policy; lists them |
| `secret_unavailable`, `forge_unavailable` | 422    | Create, patch, or enable names a retired or unknown secret or forge              |

Retiring a secret is refused while an enabled credential names it (`store/internal/secret_referrers.py` `_REFERRING`
gains `credentials`). Retiring a forge is refused while an enabled credential names it (`forge_referrers.py`).

Repositories and work sources:

```python
# wire/repository.py, wire/work_source.py — additive (bzh:released-surface-additive)
RepositoryCreateRequest.credential: IdOrSlug | None       # exactly one of credential, secret_name
RepositoryPatchRequest.credential: IdOrSlug | None
WorkSourceCreateRequest.credential: IdOrSlug | None       # required for PROVIDERS_NEEDING_CREDENTIAL
WorkSourcePatchRequest.credential: IdOrSlug | None
RepositorySummary / WorkSourceSummary += credential_id, credential_slug, credential_name, warnings: list[str]
```

| Refusal                          | Status | When                                                                                     |
| -------------------------------- | ------ | ---------------------------------------------------------------------------------------- |
| `credential_unavailable`         | 422    | Unknown or retired                                                                       |
| `credential_forge_mismatch`      | 422    | The credential's forge is not the record's; also a forge edit with no new credential     |
| `credential_binding_unsupported` | 422    | The work source's provider cannot use the binding                                        |
| `credential_conflict`            | 422    | An old field sent beside `credential` that is not the credential's `github.token` secret |
| `forge_capability_missing`       | 422    | The landing policy requires a capability outside `declared_offers` (`require_offered`)   |

The old fields, `secret_name` on a repository and `secret` on a work source, stay accepted. Every door, the
configuration apply included, resolves them through one domain function:

```python
# hub/domain/config/authoring.py
def credential_for_old_field(self, forge_id: ForgeId, secret_name: str, *, door: Door) -> tuple[CredentialId, list[str]]: ...
    # the credential with authority secret/<secret_name> on the forge, created if absent; and the warnings to answer
    # a retired one is refused 422 credential_unavailable, naming it, so the operator enables it rather than a door
```

| A write carries                                 | Result                                                                                                                                                                                                         |
| ----------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `credential` only                               | Uses it                                                                                                                                                                                                        |
| `credential` and an agreeing old field          | Uses the credential                                                                                                                                                                                            |
| An old field equal to the record's current read | No change                                                                                                                                                                                                      |
| Any other old field only                        | Resolves to the credential with authority `secret/<name>` on the record's forge, created as the migration creates one if absent; `Deprecation` header, a `warnings` entry, and a `config-old-field-used` event |

Two writes creating one credential race at `UNIQUE (tenant_id, forge_id, authority)`; the insert runs in a savepoint and
the loser re-reads the winner's row. Reads carry the old field derived from the credential, marked deprecated in
`openapi/hub.openapi.json`.

CLI, `hub/cli/credential.py`, registered as `"credential"` in `hub/cli/__init__.py`:

```text
blizzard hub credential create <slug> --forge F --binding github.token --secret S [--variant classic|fine-grained]
blizzard hub credential create <slug> --forge F --binding github.app-installation --secret S \
    --app-id N --installation-id N
blizzard hub credential list | show <c> | edit <c> [--slug] [--name] [--secret] [--variant] [--app-id] [--installation-id]
blizzard hub credential retire <c> | enable <c>
blizzard hub repo create|edit ... --credential C        # beside --secret, which resolves as the API's old field
blizzard hub source create|edit ... --credential C
```

`contracts/cli/hub.json` is regenerated.

Configuration document version 3, in `wire/config.py`:

```python
class CredentialEntry(BaseModel):          # extra="forbid"
    slug: Slug; name: str | None; forge: IdOrSlug; binding: str; secret: SecretNameStr; settings: dict[str, Any] = {}
class ConfigDocumentV3(BaseModel):         # extra="forbid"
    version: Literal[3]
    # v2's sections, in v2's order, with `credentials: list[CredentialEntry]` after `secrets`;
    # RepositoryEntryV3 and WorkSourceEntryV3 carry `credential` where v2's carry secret_name and secret
    # a v3 export of a referrer whose credential_id is NULL, which exists only before the carry-over runs,
    # writes the credential credential_for_old_field would resolve it to, without creating it: export never writes
```

`POST /api/config/apply` takes versions 2 and 3, told apart by `version`. `GET /api/config/export` and
`GET /api/config/schema/{kind}` take `version`, defaulting to 2. `hub/domain/config/apply.py` reconciles `credentials`
after `forges` and the secrets check, before work sources and repositories, through a `PlannedCredentials` lookup; a v2
entry's old field resolves through `credential_for_old_field`.

Board, `web/projects/hub/src/app/admin/credentials/`, with the file set of `admin/forges/`:

| Screen                           | Fields                                                                                                                                                      |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Credentials list                 | slug, name, forge, binding, referrer count, retired                                                                                                         |
| Credential form                  | forge (`select`), binding (`select`, filtered to the forge's kind), secret (`select`), then `variant` (`select`) or `app_id` and `installation_id` (`text`) |
| Credential detail                | the summary, `authority` read-only, referrers as links, revisions                                                                                           |
| Repository and work-source forms | `credential` (`select`, filtered to the record's forge) replaces the secret field                                                                           |
| Secret form                      | value field becomes `ConfigFieldKind` `'secret-text'`: multi-line, masked, for a PEM key                                                                    |
