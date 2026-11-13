# How the mock fleet is composed

What the mock fleet in `blizzard-mock` shares across its mocks, how the mock forge is layered, which blizzard seams it
stands in for, and what a fixture applier could drive today. Read from the `blizzard-mock`, `blizzard`, and
`blizzard-context` checkouts in November 2026. Paths are relative to `blizzard-mock/src/blizzard_mock/` unless named.

## Shared concepts

| Concept         | Where                                  | Shared how                                                                                                                                                       |
| --------------- | -------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Serving         | `serve.py`                             | By code: every daemon binds first, so `--port 0` works, and writes its listening file atomically once serving                                                    |
| Clock           | `clock.py`, `clock_api.py`             | By code: system, fixed, and controlled clocks; a controlled clock (both `BLIZZARD_CLOCK=controlled` and `BLIZZARD_CLOCK_FENCE=1`) mounts `/_clock`. The forge keeps a duplicate Protocol in `forge/domain/clock.py` |
| Levers          | `levers.py`, `forge/domain/levers.py`, `idp/api/levers.py` | By convention only: the `/_levers` shape (`GET`, `POST /reset`, `POST /{kind}`, `DELETE /{kind}`) is common, but the mock hub and mock runner share `levers.py` while the forge (fifteen state and four action levers, scoped by repo, number, branch, or pattern) and the IdP each carry their own model |
| Ids             | `ids.py`                               | By code, for the mock hub and mock-data: prefixed ULIDs                                                                                                          |
| Control planes  | per mock                               | By convention: mock hub `/_seed/*`, `/_levers`, `/_captured`; mock runner `/_drive/*`, `/_levers`; forge `/_repos`, `/_levers`                                   |
| Bare-repo truth | `fixture_workspace/seed.py: TOY_REPOS` | By seed data only: the forge (`forge/internal/git_backend.py`, GitPython) and the fixture workspace (`fixture_workspace/internal/subprocess_git.py`) each run their own git |

## The forge's layers

- `domain/`: models, a service of about 740 lines, store Protocols, levers, checks, and rate budget. `models.py` states
  that the domain carries no GitHub wire fields.
- `internal/`: in-memory stores and the git backend.
- `api/`: FastAPI routes, `serialization.py` (GitHub REST v3 JSON), `middleware.py` (GitHub `X-RateLimit-*` headers,
  ETag and 304), and a GraphQL surface that matches the one query shape blizzard sends by regular expression.

GitHub still shows through the domain in three places: `MergeableState` is GitHub's `mergeable_state` enum, issues and
pull requests share one number sequence per repo as GitHub's do (GitLab numbers issues and merge requests apart), and
some lever kinds (`secondary_rate_limited`, `rate_budget`, Actions job rerun, `update-branch`) are GitHub concepts.

`domain/` and `internal/` are about 2,100 lines and mostly vendor-neutral: git merge, rebase, and squash, the issue and
pull request lifecycle, checks, and lever mechanics. `api/` is about 1,200 lines and GitHub-only. A GitLab surface would
reuse roughly 60 to 65 percent of the forge, after generalising the mergeability states and the shared number sequence,
and add a routes-and-serialization layer of its own.

## The seams it stands in for

Blizzard has two separate seams, each with only a `github` provider:

- **Work source**: `IWorkSource` (`blizzard:src/blizzard/hub/work_sources/source.py`), with its batch reader, annotator,
  and closer ports; providers `{"github": GithubEntry}`.
- **Forge, for delivery**: `IForgeRefs`, `IForgeChanges`, `IForgeChecks` (`blizzard:src/blizzard/hub/forge/landing.py`);
  kinds `{"github": GitHubLanding}` and `KNOWN_FORGE_KINDS = {"github"}`.

There is no `mock` provider. The mock forge is a `github` forge record whose `api_base` points at it, and it serves both
seams: its issues are the work items. `hub-host.sh` in the workspace's process-compose config applies a configuration
document naming the mock forge, a `mock-forge` secret, two `github` work sources and repositories for `blizzard/toy-api`
and `blizzard/toy-web`, and a `default` project linking both, through `blizzard hub config apply`, which is idempotent.
OAuth sign-in is the IdP mock's, not the forge's. The runner makes no forge calls of its own.

Every call blizzard makes is routed by the mock forge. The work source reads issues, their comments, labels, and a
GraphQL batch of `issueOrPullRequest`; writes labels, comments, and closes; and uses no GitHub Projects. Delivery reads
refs, compares, commits, pull requests, and check runs, and writes refs, merges, pull requests, updates, and reruns. The
fidelity gaps:

- Pull request JSON carries no `closed_at`, so blizzard's `Change.closed_at` is always empty against the mock.
- The issue list ignores `per_page`, sends no `Link` header, and leaves pull requests out.
- Check-run listing ignores paging.
- Label `color` is ignored.
- GraphQL ignores fragments and the comment cap.
- The web destinations serve pull request and commit pages, not the `tree/{branch}` links the hub builds or issue
  pages.

## What a fixture applier could drive today

- **Forge.** `POST /_repos` accepts only the `toy-api` and `toy-web` seeds and refuses an existing repo. Issues,
  comments, labels, and pull requests are created through their vendor routes (with a non-GitHub `user` field), and
  state set by `PATCH`. Creating an issue always takes the next number, creating an existing label is refused, and there
  is no create-by-number or bulk import, so nothing is upsert-shaped. All issue, pull request, comment, label, check, and
  lever state is in memory: a forge restart keeps the bare repos and loses the rest.
- **Fixture workspace.** `mint`, `destroy`, `reset`, `path`. `mint` refuses an existing workspace and always mints the
  fixed toy repos.
- **mock-data.** `reset`, the `create` verbs, `scenario board|fleet`. Writes are plain inserts through schema reflection,
  behind the drift guard, so nothing upserts. `fixture list|apply` raise not-implemented.
- **Mock hub.** Its `/_seed/*` tenants, projects, scopes, and system artifacts are insert-or-replace, in memory.

Reconciliation therefore has no foothold yet in any holder of state. Each needs a way to find what a fixture already
made, by a key the fixture names, and the forge needs its state to outlive a restart or an apply must be rerun after
every one.

## Constraints from blizzard-context

- `architecture/system-shape/pluggable-seams.md`: every external system sits behind a Protocol seam chosen by
  configuration, and tests bind the mock fleet; a second vendor is a new provider in blizzard as well as a new mock.
- `architecture/system-shape/seam-positions.md`: the three landing Protocols stay separate, work items reach the forge
  through the work-source family, and each repository record names its own forge.
- `verification/blizzard/service-tests/state.md` (`bzh:service-own-mock-state`): a test scopes forge levers to its own
  repo and resets them only on a hub of its own, so an applier must never reset shared forge state under running tests.
- `tooling/store-seeding.md`: never seed the hosted deployment; a hub reset strands the runner under a new id; a running
  runner rewrites its store each tick, so seeded leases are dormant; seeded runner rows carry the runner's id and take
  outbox sequence numbers from `outbox_counters`; drift errors are never worked around with raw SQL.
- `verification/blizzard/service-tests/fixtures.md`: Playwright service specs take their fixtures only from
  `tests/harness/fixtures.ts`, a separate surface from these fixtures.
