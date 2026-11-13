# Forge vendors beyond GitHub

What it would take for the mock fleet to stand in for a second forge vendor, GitLab first, and whether a real forge in a
container could stand in instead. Surveyed November 2026. Findings only; the choice belongs to
`epic:advanced-delivery/gitlab`.

## Real GitLab CE in a container

- **Footprint.** The smallest documented setup is 2 GB RAM plus 1 GB swap and one core, with Puma workers at 0, Sidekiq
  concurrency lowered, and Prometheus off; measured at about 1.7 of 1.9 GiB used, with degraded function warned of
  (https://docs.gitlab.com/omnibus/settings/memory_constrained_envs/). Older official guidance says 4 GB minimum
  (https://archives.docs.gitlab.com/16.1/ee/install/requirements.html). The image wants `--shm-size 256m`
  (https://docs.gitlab.com/install/docker/configuration/).
- **Startup.** Undocumented beyond "may take a long time", and reconfiguration reruns on every start
  (https://docs.gitlab.com/install/docker/installation/). Reports range from minutes to over twenty
  (https://forum.gitlab.com/t/on-prem-gitlab-container-initialisation-delay/119620). Health can report ready before
  reconfiguration finishes (https://gitlab.com/gitlab-org/gitlab/-/issues/207599). Not measured here.
- **Headless seeding.** The root password is set on first boot only (`GITLAB_ROOT_PASSWORD` or
  `gitlab_rails['initial_root_password']`). A token of a known value is minted by `gitlab-rails runner` calling
  `set_token` on a new personal access token, the pattern GitLab documents for automation; tokens must expire, at most a
  year out on Free (https://archives.docs.gitlab.com/16.5/ee/user/profile/personal_access_tokens.html). Projects, issues,
  and merge requests then go through the REST API.
- **License.** CE is MIT (https://gitlab.com/gitlab-org/gitlab-foss).
- **Fit.** Too heavy and slow to boot for a per-env service or a per-pull-request gate. Plausible as a scheduled
  contract tier against a pre-seeded, snapshotted volume.

## Gitea and Forgejo in a container

- **Footprint.** One Go binary; around 200 MB idle by third-party figures, seconds to start (not measured here).
- **Headless seeding.** `GITEA__security__INSTALL_LOCK=true`, then `gitea admin user create --admin --access-token` or
  `gitea admin user generate-access-token --raw`.
- **API.** GitHub-inspired, not GitHub-compatible: everything under `/api/v1` from Gitea's own Swagger spec, commit
  statuses but no Checks API (https://plugins.jenkins.io/gitea-checks/), a non-GitHub `warning` status state, no API
  auto-merge, and merges that can be refused while mergeability recomputes after a push. No GraphQL was found (absence
  not verified). Neither offers a GitLab-compatible API.
- **Fit.** A third dialect. It adds fidelity to neither GitHub nor GitLab.

## Existing GitLab mocks and specs

- No maintained Python fake for GitLab exists. Projects stub at the HTTP client (respx, responses) or carry small
  private mocks, such as the Express mock in a third-party GitLab MCP server, which does simulate rate limiting.
- GitLab publishes an OpenAPI 3.0 spec generated from its API code (https://docs.gitlab.com/api/openapi/); an open issue
  counts 319 endpoints missing from it (https://gitlab.com/gitlab-org/gitlab/-/issues/486493).
- Spec-driven stub servers such as Prism serve that spec's shapes with canned or generated data and hold no state, so
  "merge, then the merge request reads merged" is beyond them (https://stoplight.io/open-source/prism).

## Stateful GitHub emulators

- `vercel-labs/emulate` (Apache-2.0, TypeScript) is stateful across repos, issues, pull requests with merges and branch
  protection, labels, git data, webhooks, Actions, and check runs, seeds from YAML, and resets. It has no GraphQL, no
  fault injection, no enforced rate limit, and no GitLab
  (https://cdn.jsdelivr.net/gh/vercel-labs/emulate@main/README.md). Its YAML seeding is prior art for fixture files.
- `fakehub` (Rust) is immature (https://github.com/h1alexbel/fakehub).

## Faults in front of a real forge

A real forge cannot be told to rate-limit or reject a token on demand. Toxiproxy, the common fault proxy, works at TCP
(latency, resets, bandwidth) and cannot answer HTTP 429 or 401 (https://github.com/Shopify/toxiproxy). Toxy, which did
HTTP faults, is archived. HTTP-level levers in front of a real forge would be a small proxy of the mock fleet's own, or
mitmproxy (not surveyed).

## Trade-offs

| Option                                     | Determinism                    | Edge-state levers                                         | Fidelity                              | Footprint             | Upkeep                                    |
| ------------------------------------------ | ------------------------------ | --------------------------------------------------------- | ------------------------------------- | --------------------- | ----------------------------------------- |
| Hand-written mock per vendor, shared core  | High                           | Any state, built in                                       | What is built; drifts without a guard | Small                 | High per vendor, lowered by a shared core |
| Real GitLab CE container                   | Medium: background jobs settle | Conflicts and stale branches by git; HTTP faults by proxy | Exact                                 | 2–4 GB, minutes       | Little code, much operation               |
| Gitea or Forgejo container                 | Medium                         | As GitLab                                                 | Neither vendor's                      | Hundreds of MB, fast  | Low, but a third dialect                  |
| Spec-driven stub server                    | High                           | Canned per-request answers only                           | Shapes only, no workflow              | Small                 | Low, stateless                            |
| `vercel-labs/emulate` (GitHub only)        | High                           | None                                                      | Good REST, no GraphQL                 | Small                 | External dependency                       |

The pattern the findings point to: a hand-written GitLab surface over a vendor-neutral forge core as the tier every
test and fixture uses, response shapes checked against GitLab's OpenAPI spec the way the mock hub is checked against
blizzard's, and a scheduled contract tier against a real, pre-seeded GitLab CE to catch drift in behaviour the spec does
not describe.

## Not verified

- GitLab CE's startup time and idle memory on current versions.
- Gitea's lack of GraphQL, its merge request body, and its update-branch endpoint.
- Whether `GITLAB_ROOT_PASSWORD` is still documented on the current Docker pages.
