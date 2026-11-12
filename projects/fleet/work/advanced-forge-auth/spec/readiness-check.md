# Readiness check and gates

Paths are relative to `blizzard/src/blizzard/`.

## Domain

```python
# hub/forge/readiness.py
class CheckOutcome(StrEnum):
    PROVEN = "proven"            # the read landing or ingestion makes answered 2xx
    GRANTED = "granted"          # a write the credential holds a readable grant for; not exercised
    UNPROVEN = "unproven"        # a write with no readable grant (fine-grained tokens expose none)
    REFUSED = "refused"          # carries a ForgeDiagnostic
    UNSUPPORTED = "unsupported"  # outside authorization.offered; no call made

@dataclass(frozen=True)
class OperationResult:
    operation: str               # "access" or a Capability value
    outcome: CheckOutcome
    detail: str | None           # e.g. "0 check runs on main@3f2a9c1: the read is proven, not that checks run"
    diagnostic: ForgeDiagnostic | None

@dataclass(frozen=True)
class UseResult:
    kind: Literal["repository", "work_source"]; id: str; slug: str
    operations: tuple[OperationResult, ...]
    reached: bool                # False when the 60 s cap ended the check first

@dataclass(frozen=True)
class CredentialCheck:
    credential: str; binding: str; variant: str | None
    variant_contradicted: bool   # a declared variant the token's prefix contradicts
    mint: OperationResult | None # github.app-installation only
    granted_permissions: Mapping[str, str] | None
    uses: tuple[UseResult, ...]
    not_evaluated: str = "Branch protection, required reviews, and rulesets are not evaluated."
```

Operations per use:

| Use         | Operation                                                                     | Call                                                          | Outcome when it answers 2xx |
| ----------- | ----------------------------------------------------------------------------- | ------------------------------------------------------------- | --------------------------- |
| repository  | `access`                                                                      | `GET /repos/{o}/{r}`                                          | `proven`                    |
| repository  | `refs.read`                                                                   | `GET /repos/{o}/{r}/git/ref/heads/{base}`                     | `proven`                    |
| repository  | `changes.read`                                                                | `GET /repos/{o}/{r}/pulls?state=open&base={base}&per_page=1`  | `proven`                    |
| repository  | `checks.read`                                                                 | `GET /repos/{o}/{r}/commits/{base tip}/check-runs?per_page=1` | `proven`                    |
| repository  | `changes.open`, `changes.merge.*`, `changes.update-from-base`, `checks.rerun` | none                                                          | `granted` or `unproven`     |
| work source | `issues.read`                                                                 | the sweep's issue list read, one page                         | `proven`                    |
| work source | label, comment, close                                                         | none                                                          | `granted` or `unproven`     |

A repository is checked only for `access` and the capabilities its landing policy requires (`ACTION_REQUIREMENTS`,
`POLICY_CAPABILITIES`). `granted` comes from a classic token's `X-OAuth-Scopes` containing `repo`, or an installation's
granted permission; otherwise a write is `unproven`.

## Interfaces

```text
POST /api/credentials/{credential}/checks      CONFIG_EDIT
     { "repositories": [IdOrSlug]?, "work_sources": [IdOrSlug]? }      empty: every enabled referrer
     200 CredentialCheck
     422 credential_not_used_by   a named use does not name the credential
blizzard hub credential check <credential> [--repo R]... [--source W]...
     prints one table per use; exit 1 when any outcome is refused or unsupported
```

The check runs through `IForgeAuthorizations.authorization(credential, ForgeScope.CHECK)` at `ForgePriority.SWEEPING`,
30 s per call, 60 s per check. Its client carries a transport that raises on any method other than `GET`, except the App
mint's `POST /app/installations/{id}/access_tokens`. Nothing is stored
(`decision:advanced-forge-auth/readiness-not-kept`). Each check records one `credential-check` event, severity `INFO`:

```json
{ "kind": "credential-check", "credential_id": "cred_…", "counts": { "proven": 7, "granted": 3, "unproven": 0, "refused": 1, "unsupported": 0 } }
```

The board's credential detail, repository detail, and work-source detail each gain a Check action that posts the check
for that scope and renders the `CredentialCheck` as a table.

Gates that refuse a known incompatibility:

| Gate                | Where                                                       | Reads                                                         | Answer                                                                                                                                           |
| ------------------- | ----------------------------------------------------------- | ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| Configuration write | `require_offered`, `hub/domain/config/authoring.py`         | the credential's `declared_offers`                            | `422 forge_capability_missing`, naming the capability and credential                                                                             |
| Credential edit     | `edit_credential`                                           | `declared_offers` after the edit, per enabled referrer        | `422 forge_capability_missing`, listing the referrers                                                                                            |
| Hub start           | `hub/app.py`, after live config loads                       | each enabled `github.token` credential with no `variant`      | one `WARNING` log line per credential whose effective variant withholds a capability an enabled referrer requires, naming the variant to declare |
| Pass start          | `TurnRunner._missing_capability`, `hub/delivery/flow_turn.py` | `ForgeRegistry.offered(forge, repository)`, the runtime offer | `forge-capability-missing` escalation, unchanged, with `credential` added to its event detail                                                    |

## Open questions

- Which reads the work source's GraphQL query needs, and so the `work-source` scope's permissions beyond `issues: write`
  and the check's `issues.read` call; measured against `GitHubWorkSource` when the `github-app-installations` slice
  lands. Engineering answers.
