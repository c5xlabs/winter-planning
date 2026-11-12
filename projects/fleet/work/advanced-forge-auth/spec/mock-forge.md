# Mock forge App support

Paths are relative to `blizzard-mock/src/blizzard_mock/forge/`. Every addition is off by default; no existing test
changes.

## Domain

```python
# domain/apps.py
@dataclass
class MockApp:
    app_id: int
    public_key_pem: str

@dataclass
class MockInstallation:
    installation_id: int
    app_id: int
    repositories: set[str]               # "owner/repo"
    permissions: dict[str, str]          # GitHub's names: {"contents": "write", "checks": "read", ...}
    suspended: bool = False

@dataclass
class MockToken:
    value: str                           # "ghs_…" minted, or a registered PAT
    kind: Literal["classic", "fine-grained", "installation"]
    installation_id: int | None
    permissions: dict[str, str]          # installation tokens: the narrowed set
    scopes: tuple[str, ...]              # classic: X-OAuth-Scopes
    expires_at: datetime | None
    revoked: bool = False
```

## Interfaces

New routes, in `api/apps.py`:

```text
GET  /app/installations/{id}                 Bearer JWT, RS256 over the App's key; iss, iat, exp <= 10 min checked
     200 {id, app_id, permissions, suspended_at}   401 bad JWT   404 unknown installation
POST /app/installations/{id}/access_tokens   Bearer JWT; body {permissions?}
     201 {token, expires_at, permissions}          401 bad JWT   404 unknown   403 suspended
     422 a requested permission the installation was not granted
GET  /installation/repositories              token: installation token
```

Levers, in `domain/levers.py`, set through the existing `/_levers` route:

| Lever                                         | Effect                                                                                      |
| --------------------------------------------- | ------------------------------------------------------------------------------------------- |
| `STRICT_TOKENS`                               | Only registered PATs and minted tokens authenticate; anything else is `401 Bad credentials` |
| `REGISTER_TOKEN`                              | Registers a PAT with `kind` and `scopes`                                                    |
| `REGISTER_APP`, `REGISTER_INSTALLATION`       | Registers an App's id and public key, or an installation                                    |
| `TOKEN_LIFETIME`                              | Seconds a minted token lives; default 3600                                                  |
| `REVOKE_TOKEN`                                | The token answers 401 from then on                                                          |
| `SUSPEND_INSTALLATION`, `DELETE_INSTALLATION` | Mint answers 403 or 404; minted tokens answer 401                                           |
| `REMOVE_REPOSITORY`                           | The installation's tokens get 404 on that repository                                        |
| `SET_PERMISSIONS`                             | Replaces an installation's granted permissions                                              |
| `PRIVATE_REPOSITORY`                          | Marks a repository private                                                                  |

Under `STRICT_TOKENS`:

| Request                                                        | Answer                                                                                                   |
| -------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| Installation token on a route its permissions do not cover     | `403`, `X-Accepted-GitHub-Permissions: <name>=<level>`, message "Resource not accessible by integration" |
| Fine-grained PAT on `GET …/check-runs` of a private repository | `403`, message "Resource not accessible by personal access token"                                        |
| Classic PAT                                                    | every route; `X-OAuth-Scopes` echoes its scopes                                                          |
| Any request                                                    | `X-GitHub-Request-Id` set                                                                                |

The fine-grained row follows `FINE_GRAINED_WITHHELD`; when `decision:advanced-forge-auth/fine-grained-checks` changes,
the row changes with it.
