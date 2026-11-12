# Forge refusals

Paths are relative to `blizzard/src/blizzard/`.

## Domain

```python
# hub/forge/landing.py — beside ForgeRejected; imports no httpx
class AuthRefusal(StrEnum):
    CREDENTIAL_INVALID = "credential-invalid"          # wrong, expired, or revoked token
    PERMISSION_MISSING = "permission-missing"
    TOKEN_UNSUPPORTED = "token-unsupported"            # the token variant cannot make this call
    SSO_REQUIRED = "sso-required"
    KEY_REJECTED = "key-rejected"                      # App JWT refused: wrong key, wrong app id, or clock skew
    INSTALLATION_MISSING = "installation-missing"
    INSTALLATION_SUSPENDED = "installation-suspended"

@dataclass(frozen=True)
class ForgeDiagnostic:
    status: int
    operation: str                   # seam operation or work-source call: "checks", "merge", "close", "mint"
    endpoint: str                    # method and route template, never a filled-in URL or query
    repository: str | None           # "owner/repo"
    message: str                     # sanitize_forge_text(body["message"])
    documentation_url: str | None
    request_id: str | None           # X-GitHub-Request-Id
    accepted_permissions: str | None # X-Accepted-GitHub-Permissions
    credential: str | None           # slug
    binding: str | None
    variant: str | None

class ForgeAuthRefused(ForgeRejected):     # reason stays ForgeRejected.AUTH; existing reason checks are unchanged
    refusal: AuthRefusal
    diagnostic: ForgeDiagnostic
# ForgeRejected gains `diagnostic: ForgeDiagnostic | None`, set for PROTECTED, METHOD, and message rejections

# hub/work_sources/source.py — beside WorkSourceError
class WorkSourceAuthRefused(WorkSourceError):
    refusal: AuthRefusal
    diagnostic: ForgeDiagnostic
```

```python
# hub/forge/diagnostics.py
def sanitize_forge_text(text: str) -> str: ...
    # removes gh[pousr]_\w+, github_pat_\w+, JWTs (eyJ…\.…\.…), PEM blocks, and Authorization values;
    # collapses whitespace; truncates to 500 characters

# hub/forge/internal/github_refusals.py — shared by GitHubLanding, GitHubWorkSource, and InstallationTokens
def classify_auth_refusal(response: httpx.Response, *, operation: str, endpoint: str,
                          repository: str | None, authorization: ForgeAuthorization | None) -> ForgeAuthRefused: ...
```

## Interfaces

`classify_auth_refusal` runs on a 401 or 403 after `github_rates.py` has split off rate limits:

| Answer                                                                   | Refusal                                                                                   |
| ------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------- |
| 401 on a request                                                         | `credential-invalid`                                                                      |
| 403 with `X-GitHub-SSO: required`, or a message naming SAML enforcement  | `sso-required`                                                                            |
| 403 on an operation whose capability the effective variant withholds     | `token-unsupported`                                                                       |
| Any other 403                                                            | `permission-missing`                                                                      |
| Mint or installation read: 401 / 404 / 403 with `suspended_at` set / 422 | `key-rejected` / `installation-missing` / `installation-suspended` / `permission-missing` |

Unchanged: `rerun_check` answers 401, 403, 404, and 422 as `Rerun(REFUSED)` and never raises; a 404 still probes the
repository once. For an App credential, the `ForgeNotFound(REPOSITORY)` detail adds "or the installation does not
include it". A 5xx body is sanitized into `ForgeUnavailable`'s message.

`forge-auth` escalation detail, built by `RepoLanding.ask` (`hub/delivery/actions/landing/repo.py`):

```text
<owner/repo> on forge <forge slug>: <operation sentence> was refused: <refusal sentence>.
GitHub said: "<message>" (HTTP <status>, request <request_id>).
Credential <slug> (<binding>, <variant>). <remedy>
```

| Refusal                  | Remedy                                                                                     |
| ------------------------ | ------------------------------------------------------------------------------------------ |
| `credential-invalid`     | Replace the token, or check the App's installation                                         |
| `permission-missing`     | Grant `<accepted_permissions>` to the token or the App installation                        |
| `token-unsupported`      | Use a classic token or a GitHub App, or a landing policy that does not need `<capability>` |
| `sso-required`           | Authorize the token for the organization's SSO                                             |
| `key-rejected`           | Replace the App's private key, or correct its App id, or check the hub's clock             |
| `installation-missing`   | Reinstall the App, or correct the installation id                                          |
| `installation-suspended` | Unsuspend the App's installation                                                           |

The readiness-and-recovery slice appends
`Then run: blizzard hub credential check <slug> --repo <repo>; blizzard hub
chunk requeue <chunk>.` Before a credential
exists for the repository, `Credential …` names the repository's secret.

Event `forge-auth-refused`, joining `foundation/event_log.py`'s kinds at severity `WARNING`, recorded once per
escalation and once per refused work-source sweep:

```json
{ "kind": "forge-auth-refused", "chunk_id": "ch_…|null", "repository_id": "repo_…|null", "work_source_id": "wsrc_…|null",
  "forge_id": "frg_…", "refusal": "token-unsupported", "diagnostic": { "...ForgeDiagnostic fields" } }
```

`GitHubWorkSource` (`hub/work_sources/internal/github_work_source.py`) raises `WorkSourceAuthRefused` where it raises a
bare `WorkSourceError(str(exc))` for a 401 or 403 today; its reporters print the diagnostic's refusal sentence and
remedy. `SeamCommitResolver` (`hub/forge/commit_check.py`) still answers any failure as `None`, and logs a
`ForgeAuthRefused` diagnostic at `WARNING`.

No diagnostic, log line, event, escalation, or API answer carries a token, App JWT, private key, or `Authorization`
value (`engineering:secrets-out-of-evidence`).
