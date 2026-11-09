# The mock's GitLab face

Paths are relative to `blizzard-mock`.

## Interfaces

`blizzard-mock`'s forge gains a GitLab face: `/api/v4` routes for every operation in [adapter.md](./adapter.md), over
the same git repositories its GitHub face serves, so one mock instance hosts both kinds and a chunk can span them. Its
answers carry GitLab's `RateLimit-*` headers.

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
