# Where a GitLab repository lives

Paths are relative to `blizzard/src/blizzard/`.

## Domain

A GitLab project's owner is a path of nested groups (`group/subgroup`). A GitLab repository record's `owner` holds the
full namespace.

```python
# hub/domain/config/repositories.py: validate_fields
# owner may hold "/" when the record's forge kind is "gitlab"; on a "github" record, whose owner is one segment, a "/"
# in owner is refused ConfigFieldError("owner", ...).
```

```python
# foundation/repo_ref.py (cases in tests/unit/foundation/test_repo_ref.py)
@dataclass(frozen=True)
class RepoRef:
    host: str
    owner: str      # the immediate parent segment, as today
    name: str
    namespace: str  # new: every segment between host and name, "/"-joined

# repo_identity and pointer_key key by the whole namespace: "a/x/r" and "b/x/r" on one host are two repositories.
# A GitHub origin's namespace is its one owner, so its key is unchanged (engineering:github-unchanged).
# Both run on the runner (runner/lifecycle/judgement/git_commits.py) as well as the hub
# (engineering:runner-hub-release).
```

```python
# hub/domain/config/repositories.py
@dataclass(frozen=True)
class CommitOrigin:
    name: str
    host: str = ""
    owner: str = ""
    namespace: str = ""  # new, from RepoRef.namespace

def _candidates(origin, rows, forges) -> list[ConfiguredRepository]:
    # A "gitlab" record's owner matches origin.namespace (whole path, case-insensitive);
    # a "github" record's owner matches origin.owner (one segment), as today.
    ...
```

| Origin                             | `repo_identity` before | `repo_identity` after | `pointer_key` after    |
| ---------------------------------- | ---------------------- | --------------------- | ---------------------- |
| `https://github.com/o/r.git`       | `o/r`                  | `o/r`                 | `github.com/o/r`       |
| `git@github.com:o/r.git`           | `o/r`                  | `o/r`                 | `github.com/o/r`       |
| `https://gitlab.example/a/x/r.git` | `x/r`                  | `a/x/r`               | `gitlab.example/a/x/r` |
| `https://gitlab.example/b/x/r.git` | `x/r`                  | `b/x/r`               | `gitlab.example/b/x/r` |

## Interfaces

```python
# hub/domain/config/forges.py: derived_web_origin gains a "gitlab" branch
def derived_web_origin(kind: str, api_base: str) -> str:
    stripped = api_base.rstrip("/")
    if kind == "gitlab" and stripped.endswith("/api/v4"):
        return stripped[: -len("/api/v4")]
    ...  # "github" branches unchanged

# hub/forge/web.py: web_links(forge) answers a GitLab binding for kind "gitlab"
class GitLabWebLinks:  # IForgeWebLinks
    web_origin: str

    def branch_url(self, repository: RepositoryFields, branch_name: str) -> str:
        return f"{self.web_origin}/{repository.owner}/{repository.repo}/-/tree/{branch_name}"

    def commit_url(self, repository: RepositoryFields, sha: str) -> str:
        return f"{self.web_origin}/{repository.owner}/{repository.repo}/-/commit/{quote(sha, safe='')}"

# A merge request's address: {web_origin}/{owner}/{repo}/-/merge_requests/{iid}
```

The board links a GitLab chunk's commits and merge request from the repository's forge: `delivery_read.py`'s
`_commit_url`, which parses a GitHub pull request's address, asks `web.py` for a commit's address instead.

## Data

No table or column changes and no migration. A GitHub origin keys exactly as before, so no stored pointer or marker
written before the epic changes meaning.

## Open questions

- How `validate_fields(fields)` learns the record's forge kind; it takes no forge today. Engineering.
- What `CommitOrigin.of(None, name)` sets `namespace` to for a stored name with no origin, which today splits `owner`
  off the last `/`. Engineering.
- Whether `_commit_url` loses its GitHub-address parsing, or stays GitHub-only while GitLab commits take
  `DeliveryRead.of`'s existing `commit_link` fallback (`LandedCommitLinks`, through `web_links`); and whether
  `IForgeWebLinks` gains a merge-request address or the board takes it from `Change.web_url`. Engineering.
- The class name `GitLabWebLinks` follows `GitHubWebLinks`; confirm it. Engineering.
