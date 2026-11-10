# Shards

Paths are relative to the `blizzard` repo root.

## Interfaces

### The recorder

```python
# tests/support/suite_plugin.py — engineering:balanced-shards, engineering:gate-measured

#: Names the file a run writes its durations to; unset, nothing is recorded and no shard's membership changes.
DURATIONS_OUT_ENV = "BLIZZARD_DURATIONS_OUT"


def pytest_runtest_logreport(report: pytest.TestReport) -> None:
    """Adds ``report.duration`` of the setup, call, and teardown phases to ``report.nodeid``'s total.

    A skipped test is not recorded: its time says nothing about its cost when it runs.
    """


def pytest_sessionfinish(session: pytest.Session) -> None:
    """Under xdist each worker writes its own part, ``<file>.<worker id>``, and the controller merges the parts into
    ``<file>``, so the file holds every test the run executed exactly once. A red run writes its file too."""
```

### The split

```python
# tests/support/suite_plugin.py — engineering:balanced-shards

DURATIONS_TABLE = repo_root() / "tests" / "durations.json"


def partition(nodeids: Sequence[str], durations: Mapping[str, float], count: int) -> dict[str, int]:
    """Each nodeid's 1-based shard, by longest-processing-time packing.

    Computed once per collection. Every shard, and every xdist worker in it, computes the same partition from the same
    collection and table, so every test lands in exactly one shard, in any item order.
    """
    fallback = statistics.median(durations.values())  # a test the table lacks is placed as an ordinary one, not as free
    cost = {nodeid: durations.get(nodeid, fallback) for nodeid in nodeids}
    totals = [0.0] * count
    shards: dict[str, int] = {}
    for nodeid in sorted(nodeids, key=lambda n: (-cost[n], n)):  # duration descending, ties by nodeid
        index = min(range(count), key=lambda i: (totals[i], i))  # smallest running total, ties to the lowest index
        shards[nodeid] = index + 1
        totals[index] += cost[nodeid]
    return shards


@pytest.hookimpl(tryfirst=True)
def pytest_collection_modifyitems(config: pytest.Config, items: list[pytest.Item]) -> None:
    apply_folder_layout(items)
    refuse_unmarked(items)
    _retire_aged_revision_tests(items)
    spec = os.environ.get(SHARD_ENV)
    if not spec:
        return  # a local `uv run pytest` deselects nothing
    # BLIZZARD_TEST_SHARD_BY=file: Open questions
    index, count = parse_shard(spec)
    table = _read_table(DURATIONS_TABLE)  # None when missing or unreadable
    if table is None:
        # warns, then keeps today's split: the gate never fails for want of times
        keep = {item.nodeid for item in items if shard_of(item.nodeid, count) == index}
    else:
        shards = partition([item.nodeid for item in items], table, count)
        keep = {nodeid for nodeid, shard in shards.items() if shard == index}
    # deselect every item not in `keep`, through config.hook.pytest_deselected, as today
```

The table orders the tests and sums the totals only. A shard still runs `-n auto`, and xdist schedules within it as it
does today. The service tier's pytest half splits through the same hook, from its own nodeids in the same table.

### The refresh script

```text
scripts/test-durations.py refresh        # also `mise run test-durations`

1. gh run download the `durations-*` artifacts of the last 10 successful `push` runs on `master`
2. per nodeid, the median over the runs that executed it, rounded to 0.01 s
3. drop nodeids no longer collected
4. write tests/durations.json, keys sorted
```

## CI

```yaml
# .github/workflows/gate.yml — engineering:balanced-shards, engineering:gate-measured
python-tests:
  strategy:
    fail-fast: false
    matrix:
      shards: [6]
      shard: [1, 2, 3, 4, 5, 6]
  steps:
    # … checkout, uv, uv sync, as today
    - name: pytest
      env:
        BLIZZARD_TEST_SHARD: ${{ matrix.shard }}/${{ matrix.shards }}
        BLIZZARD_TEST_SHARD_BY: file
        BLIZZARD_DURATIONS_OUT: durations.json
      run: uv run pytest -n auto
    - name: keep the durations
      if: ${{ !cancelled() }} # a red run's times are kept too
      uses: actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02 # v4.6.2
      with:
        name: durations-python-tests-${{ matrix.shard }}
        path: durations.json
        retention-days: 14
```

```yaml
# .github/workflows/upper-tiers.yml — the pytest half only; service-playwright is unchanged
service-tier:
  steps:
    # … as today
    - name: service tier — a running daemon's HTTP API vs. its mocked counterpart
      env:
        BLIZZARD_SERVICE: "1"
        BLIZZARD_TEST_SHARD: ${{ matrix.shard }}/${{ matrix.shards }}
        BLIZZARD_DURATIONS_OUT: durations.json
      run: cd blizzard && uv run pytest -n auto tests/service/ -v
    - name: keep the durations
      if: ${{ !cancelled() }}
      uses: actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02 # v4.6.2
      with:
        name: durations-service-tier-${{ matrix.shard }}
        path: blizzard/durations.json
        retention-days: 14
```

```yaml
# .github/workflows/durations.yml — new; drift is noticed without anyone watching for it
name: durations
on:
  schedule:
    - cron: "<weekly>" # schedule unsettled (Open questions)
jobs:
  refresh:
    runs-on: ubuntu-latest
    steps:
      # … checkout, uv, uv sync
      - run: uv run python scripts/test-durations.py refresh
      # opens a pull request only when the new table moves any shard's total by more than 5%
```

An ordinary change may refresh the table at any time.

## Data

```jsonc
// durations.json — one run's output, uploaded as durations-<job>-<shard>
{ "<nodeid>": 12.34 }  // seconds: setup + call + teardown

// tests/durations.json — the committed table
{ "<nodeid>": 12.34 }  // seconds: median over the runs it was built from, rounded to 0.01, keys sorted
```

## Open questions

- Whether the split packs test files or tests. `gate.yml`'s `python-tests` now shards by file
  (`BLIZZARD_TEST_SHARD_BY=file`) in `pytest_ignore_collect`, so a shard imports only its own modules, and a per-test
  partition in `pytest_collection_modifyitems` would import every module in every shard. Answered when the spec is taken
  again against the current gate (`engineering:reground-before-build`). — engineering
- `durations.yml`'s weekly schedule, how it opens its pull request, and with what token: a pull request opened with the
  default `GITHUB_TOKEN` starts no `pr.yml` run. — engineering
