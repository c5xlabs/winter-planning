# Slow outliers

Paths are relative to the `blizzard` repo root.

## The measurement

```yaml
# .github/workflows/gate.yml — engineering:outlier-triage
on:
  workflow_call:
  workflow_dispatch:
    inputs:
      <durations-input>: # name and type unsettled (Open questions)
        description: Add --durations=0 to the unit and component shards.

jobs:
  python-tests:
    runs-on: ubuntu-latest
    steps:
      - name: pytest
        run: uv run pytest -n auto ${{ inputs.<durations-input> && '--durations=0' || '' }}
```

After the measurement, the durations recorder ([shards.md](./shards.md#the-recorder)) gives the same numbers on every
run.

## Sharing the fake binary

```python
# tests/component/runner/harness/opencode/compatibility/test_diagnostic.py — engineering:outlier-triage
@pytest.fixture(scope="module")
def fake_opencode(tmp_path_factory: pytest.TempPathFactory) -> str:
    """Written once per module, where `_probe()` writes the two-line `fake-opencode` into each test's `tmp_path` today.

    No test writes to or executes it, so sharing it changes no test's subject. A test found to modify or execute it
    keeps its own `_stub_binary`.
    """
    return _stub_binary(tmp_path_factory.mktemp("bin"))


def _probe(tmp_path: Path, *, binary: str, process: IOpenCodeProcess | None = None, ...) -> OpenCodeCompatibilityProbe:
    ...  # binary=binary, the rest as today
```

## Open questions

- Where the cost of `test_diagnostic.py` lies. It was recorded at about 164 s of CPU, one 35 s test and some thirty at
  4–5 s, all spawning subprocesses, but the file as it stands holds twenty test functions, twenty-four items with one
  parametrized five ways, and its subprocess tests carry bounds of 5 s and fixed sleeps of at most 0.7 s. The cost is
  either in what the tests start (the pty fork, the Landlock probe, the six megabytes each pipe drains, the
  `CompatibilityDiagnostic` runs) or in a recent change that already cut it. The measurement answers it. — engineering
- The `workflow_dispatch` input's name and type. — engineering
