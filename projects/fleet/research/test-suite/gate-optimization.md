# Remaining gate cost

Investigation of remaining CI cost against `blizzard` at `4645daa28326c169fd4468ec6dcb0acdc4ec9bb6`, using successful
GitHub Actions runs on 14–15 November 2026. This is a source sample, not a performance target or a guarantee of savings.

## Normal pull requests

Elapsed wait is from run creation to the last successful required check, including the aggregate result jobs. Hotfix
runs are excluded. The conditional store proof ran on the live-settings PR but did not determine its finish.

| Run                                                                            | Slowest Python shard | Post-shard Python result job | Slowest Playwright service shard | Final required check after creation |
| ------------------------------------------------------------------------------ | -------------------- | ---------------------------- | -------------------------------- | ----------------------------------- |
| [43751531009](https://github.com/paul-gross/blizzard/actions/runs/43751531009) | 10m13s               | 3m53s                        | 23m09s                           | 23m30s                              |
| [43718080164](https://github.com/paul-gross/blizzard/actions/runs/43718080164) | 10m01s               | 3m49s                        | 13m09s                           | 14m11s                              |
| [43674376639](https://github.com/paul-gross/blizzard/actions/runs/43674376639) | 11m09s               | 3m38s                        | 12m27s                           | 14m51s                              |
| [43592040068](https://github.com/paul-gross/blizzard/actions/runs/43592040068) | 11m36s               | 2m56s                        | 21m02s                           | 22m03s                              |
| [43579980656](https://github.com/paul-gross/blizzard/actions/runs/43579980656) | 11m09s               | 4m10s                        | 20m54s                           | 23m00s                              |

Median complete wait: **22m03s**; maximum: **23m30s**. The service tier determines three finishes; the post-shard Python
result determines the other two. Individual job durations exclude time before their job started.

## Master pushes

| Run                                                                            | Slowest Python shard | Post-shard Python result job | Slowest Playwright service shard | Dev image publish job |
| ------------------------------------------------------------------------------ | -------------------- | ---------------------------- | -------------------------------- | --------------------- |
| [43753786351](https://github.com/paul-gross/blizzard/actions/runs/43753786351) | 11m17s               | 4m36s                        | 15m15s                           | 4m34s                 |
| [43742286391](https://github.com/paul-gross/blizzard/actions/runs/43742286391) | 10m30s               | 3m53s                        | 17m19s                           | 3m32s                 |
| [43675493205](https://github.com/paul-gross/blizzard/actions/runs/43675493205) | 12m54s               | 3m17s                        | 20m51s                           | 3m49s                 |
| [43668018494](https://github.com/paul-gross/blizzard/actions/runs/43668018494) | 9m33s                | 4m37s                        | 16m09s                           | 3m41s                 |
| [43593855656](https://github.com/paul-gross/blizzard/actions/runs/43593855656) | 10m19s               | 3m22s                        | 13m05s                           | 4m50s                 |

The bounded push crash sweep is not the full PR sweep. Dev publication follows the gate and is a separate wait, not a PR
check; cutting an image architecture cannot shorten a pull request.

In push `43753786351`, Playwright shard jobs took **313, 915, 913, 305, 312 and 178 seconds**. Shards 2 and 3 are heavy
despite the similar case counts visible in their logs: 123 and 125 passed. Native sharding is not duration balancing.

## What the code and logs show

Paths below are relative to `blizzard`.

| Source                                                                     | Observation                                                                                                           | Remaining candidate                                                                               |
| -------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `gate.yml::python-tests-result`, `tests/support/aggregate_census.py::main` | The result job reruns dynamic API observations after all shards; the latest push spent 251s in the census step        | Run that unchanged proof beside the shards, retaining the same required result check              |
| `tests/playwright.config.ts`, `upper-tiers.yml::service-playwright`        | CI already writes JSON and latency reports, but the workflow uploads them only on failure                             | Retain small successful reports for timing-based placement and triage                             |
| `tests/harness/projects.ts`                                                | Default projects share two CI workers; non-default hub/fleet profiles are limited to one                              | Model profile limits and setup cost; test worker changes rather than assuming more is faster      |
| `service/fleet/garden-routine.spec.ts`                                     | One history exercises repeated observations, proposals and recovery; 114s in the latest push and about 186s in the PR | Measure scripted CLI/host work; split only independent histories, preserving cross-run assertions |
| `service/fleet/acceptance-loop.spec.ts`                                    | The whole-chunk collector case uses a real 90s sampling window; about 108s in the latest push                         | Balance its placement; do not treat the external collector as controlled-clock work               |
| `service/runner/system-artifacts.spec.ts`                                  | Unreachable-hub case outwaits bounded retries; 58.2s in the latest push                                               | Check whether the retry duration is incidental and publicly configurable                          |
| `service/runner/opencode2{,-levers}.spec.ts`                               | Never-answering/registering service cases each took about 41s in the latest push                                      | Measure boot versus capability deadlines before shortening either                                 |
| `tests/harness/fixtures.ts::traced`                                        | Controlled-clock export barriers already exist, but several companions still use system-clock export polling          | Adopt the existing mechanism where time itself is not the subject                                 |

The two inspected heavy push shards spent 857s and about 864s in Playwright itself. Chromium installation was about 32s
on shard 3, so eliminating that installation alone would not resolve its wait. Per-case log durations are not complete
fixture profiles; their sums overlap under two workers and cannot be read as wall time.

## Existing mechanisms, not new work

- `suite_plugin.py` already excludes other gate shards' modules before import.
- `init-templates.ts` already builds per-run stores with real daemon initialization; cold-init and reset proofs remain.
- Controlled clocks, held-stack fixtures, worker checkpoints, and trace export barriers already exist.
- Normal PR UI, image-smoke and compose-smoke jobs build both boards. The frontend job and packaging jobs still have
  independent builds, but none was the longest path in this sample.
- Dev and release images already support amd64 and arm64.

The bounded changes are tracked independently:

- [blizzard#2225](https://github.com/paul-gross/blizzard/issues/2225): shorten the silent-daemon test wait.
- [blizzard#2227](https://github.com/paul-gross/blizzard/issues/2227): replace descendant-reaping sleeps.
- [blizzard#2226](https://github.com/paul-gross/blizzard/issues/2226): run the refusal census beside Python shards.

The Playwright entries above are investigation leads, not a committed scheduler or timing-infrastructure project.
Further outlier changes need measurements and preserved assertion boundaries; no broad optimization epic owns them.
