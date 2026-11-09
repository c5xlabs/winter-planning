# The mock's GitLab face

Paths are relative to `blizzard-mock`.

## System

One `blizzard-mock` forge instance serves a GitHub face and a GitLab face over the same git repositories, so one chunk
can span both kinds. Serves `engineering:gitlab-scenarios`; written from GitLab's documentation, and checked against a
real GitLab by `engineering:gitlab-contract-suite`.

## Interfaces

Every route the adapter calls ([adapter.md](./adapter.md#interfaces)), under `/api/v4`, each answer carrying
`RateLimit-Limit`, `RateLimit-Remaining`, and `RateLimit-Reset`:

| Method | Path                                                    |
| ------ | ------------------------------------------------------- |
| `GET`  | `/api/v4/version`                                       |
| `GET`  | `/api/v4/projects/:id/repository/branches/:branch`      |
| `GET`  | `/api/v4/projects/:id/repository/compare`               |
| `GET`  | `/api/v4/projects/:id/repository/merge_base`            |
| `GET`  | `/api/v4/projects/:id/repository/commits/:sha`          |
| `GET`  | `/api/v4/projects/:id/repository/commits/:sha/statuses` |
| `GET`  | `/api/v4/projects/:id/merge_requests`                   |
| `POST` | `/api/v4/projects/:id/merge_requests`                   |
| `GET`  | `/api/v4/projects/:id/merge_requests/:iid`              |
| `PUT`  | `/api/v4/projects/:id/merge_requests/:iid/rebase`       |
| `PUT`  | `/api/v4/projects/:id/merge_requests/:iid/merge`        |
| `GET`  | `/api/v4/projects/:id/pipelines/:pipeline_id/jobs`      |
| `POST` | `/api/v4/projects/:id/jobs/:job_id/retry`               |

Levers, beside the GitHub face's under `/_levers`:

| Lever                    | Produces                                                                                    |
| ------------------------ | ------------------------------------------------------------------------------------------- |
| `need_rebase`            | A merge request whose `detailed_merge_status` is `need_rebase`                              |
| Conflicting rebase       | A rebase GitLab answers with a conflict                                                     |
| Failing pipeline         | A merge request whose pipeline fails                                                        |
| Merged-results pipeline  | A pipeline that runs on a merge ref rather than the head                                    |
| Retried job              | A job that failed and was retried                                                           |
| `locked`                 | A merge request passing through `locked` on its way to `merged`                             |
| Old server               | `GET /version` reporting a version below 15.6                                               |
| Rate remaining and reset | The `RateLimit-Remaining` and `RateLimit-Reset` it answers, as the GitHub face's lever sets |

## Open questions

- The `LeverKind` names for the levers the table describes but does not name, and whether the rate lever is the existing
  `rate_budget` action lever extended to the GitLab face. Engineering.
- Which GitLab merge methods the mock's projects can be set to (merge commit, semi-linear, fast-forward), since
  `merged_revision` differs by method. Engineering.
