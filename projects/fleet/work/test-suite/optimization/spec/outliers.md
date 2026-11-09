# Slow outliers

## The measurement

`gate.yml` gains a `workflow_dispatch` input that adds `--durations=0`. The unit and component tiers run once with it on
an `ubuntu-latest` runner, and the slowest fifty items are recorded beside the baseline (`engineering:outlier-triage`).
The durations recorder ([pipeline.md](./pipeline.md#recording-durations)) gives the same numbers on every run
thereafter. The rest of this leaf applies to whatever outliers the measurement names.

## Sharing the fake binary

`_stub_binary` writes a two-line `fake-opencode` script into each test's `tmp_path` through `_probe()`, and the script
is never executed. A module-scoped fixture writes it once into a `tmp_path_factory` directory, and `_probe()` takes it
from there. Nothing a test does writes to the binary, so sharing it changes no test's subject. Any test found to modify
or execute it keeps its own.

The same applies to any other outlier the measurement names that builds an identical fixture per test: it is shared at
the widest scope at which no test mutates it.
