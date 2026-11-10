# Resolution tracing

## Domain

### Provenance vocabulary

Every resolved cell carries one source from this closed set. The JSON carries the kebab-case term; the terminal renders
it as words.

| Source             | Applies to            | Meaning                                                                                            |
| ------------------ | --------------------- | -------------------------------------------------------------------------------------------------- |
| `graph`            | session field         | The declared session supplied the field                                                            |
| `chunk`            | session field         | The declaration left the field empty, and the chunk's default supplied it                          |
| `unset`            | session field, effort | Neither layer supplied it                                                                          |
| `runner-alias`     | model, effort         | An entry in this runner's alias table for this harness                                             |
| `adapter-default`  | model                 | The binding's built-in tier table                                                                  |
| `native`           | model                 | An unprefixed, harness-native model name passed through as authored                                |
| `adapter-fallback` | model                 | Nothing in the preference resolved, and the non-strict path fell back to the binding's own default |
| `unmapped`         | tier                  | Neither the runner nor the binding maps this tier for this harness                                 |
| `as-authored`      | effort                | A well-known effort passed through unchanged                                                       |
| `dropped`          | effort                | An effort the binding does not recognize; nothing reaches the harness                              |

```python
# engineering:traced-resolution — each phase's traced form returns the value with the layer that supplied it.
# The existing entry points are projections of these and keep their signatures.
FieldSource = Literal["graph", "chunk", "unset"]
ModelSource = Literal["runner-alias", "adapter-default", "native", "adapter-fallback", "unmapped"]
EffortSource = Literal["runner-alias", "as-authored", "dropped", "unset"]


@dataclass(frozen=True)
class Traced[T]:
    value: T
    source: FieldSource
```

### The hub merge

`blizzard.hub.domain.execution.envelope`:

```python
@dataclass(frozen=True)
class EffectiveSession:
    name: str | None
    model: list[str]
    effort: str | None
    rotate: RotatePolicy | None
    compaction_window: str | None
    harnesses: list[str]
    sources: SessionSources  # new: one FieldSource each for model, effort, harnesses

    @classmethod
    def of(cls, chunk: Chunk, graph: Graph, node: Node) -> EffectiveSession:
        declaration = graph.session_by_name(node.session_source) if node.session_source else None
        return cls.merge(declaration, ChunkDefaults.of(chunk))  # signature and result unchanged

    @classmethod
    def merge(cls, declaration: SessionDecl | None, defaults: ChunkDefaults) -> EffectiveSession:
        # The declaration-level merge the report calls; `defaults` is empty when no chunk is named.
        # model, harnesses: a nonempty declared list wins whole, never merged    -> "graph"
        #                   else a nonempty chunk default                         -> "chunk", else "unset"
        # effort:           a non-null declared value wins                        -> "graph"
        #                   else a non-null chunk default                         -> "chunk", else "unset"
        # declaration None (a bare fresh, resume, or resume:<node> lineage): all three from the chunk.
        ...
```

The node envelope keeps its shape (`session_model`, `session_effort`, `session_harnesses`). Sources travel only on the
[hub read](./hub-sessions.md).

### Adapter resolution

Each binding (`ClaudeCodeAdapter`, `OpenCodeAdapter`, `OpenCodeTwoAdapter`) gains a traced form over the shared skeleton
in `blizzard.runner.harness.harness_shared`:

```python
@dataclass(frozen=True)
class ModelTrace:
    value: str | None  # None: strict and nothing resolved, or OpenCode's fallback (no --model)
    source: ModelSource
    via: str | None  # the winning preference entry
    tried: tuple[tuple[str, str], ...]  # each earlier entry, with the reason it did not resolve


@dataclass(frozen=True)
class EffortTrace:
    authored: str | None
    runner_alias: str | None
    value: str | None  # what reaches the harness
    flag: str | None  # "--effort" | "--variant"; None when nothing reaches the harness
    source: EffortSource


def trace_model(
    resolve_one: Callable[[str], ModelTrace | None], default: str | None, preferences: Sequence[str], *, strict: bool
) -> ModelTrace: ...

# Projections, unchanged signatures:
#   resolve_model_strict(prefs) == trace_model(..., strict=True).value
#   resolve_model(prefs)        == trace_model(..., strict=False).value
#   resolve_effort(v)           == trace_effort(v).value
```

Model resolution walks the preference entries left to right, for one harness:

| Entry                       | Checked, in order                                                                                | Source             |
| --------------------------- | ------------------------------------------------------------------------------------------------ | ------------------ |
| `blizzard:` tier            | The runner's model alias table for this harness                                                  | `runner-alias`     |
|                             | The binding's built-in tier table                                                                | `adapter-default`  |
|                             | Neither: the next entry is tried                                                                 | `unmapped`         |
| Unprefixed                  | The runner's model alias table for this harness                                                  | `runner-alias`     |
|                             | Matches the binding's supported names; for OpenCode, also parses as a `provider/model` reference | `native`           |
|                             | Neither: unresolved, and the next entry is tried                                                 | —                  |
| Nothing resolved, strict    | No model; the harness cannot serve the session                                                   | —                  |
| Nothing resolved, nonstrict | The binding's default                                                                            | `adapter-fallback` |

Effort is traced per harness. A value that reaches the harness is rendered as the argument it becomes.

| Authored                             | Result                                                                                                | Source         |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------- | -------------- |
| `null`                               | Nothing passed                                                                                        | `unset`        |
| A key of the runner's effort aliases | The alias value                                                                                       | `runner-alias` |
| `low`, `medium`, `high`, `max`       | Passed unchanged                                                                                      | `as-authored`  |
| Anything else                        | Nothing passed; the harness runs at its own default. Claude Code and OpenCode log it once per adapter | `dropped`      |

Per-harness tables:

| Harness       | Model aliases               | Effort aliases              | Built-in tiers                                                                           | Default when nothing resolves                         | Effort flag |
| ------------- | --------------------------- | --------------------------- | ---------------------------------------------------------------------------------------- | ----------------------------------------------------- | ----------- |
| `claude_code` | `[models.aliases]`          | `[effort.aliases]`          | `blizzard:frontier` → `fable`, `blizzard:advanced` → `opus`, `blizzard:basic` → `sonnet` | `claude-opus-5`                                       | `--effort`  |
| `opencode`    | `[opencode.models.aliases]` | `[opencode.effort.aliases]` | none                                                                                     | No `--model`; OpenCode's own configured model decides | `--variant` |

### Harness selection

`blizzard.runner.lifecycle.session`:

```python
Outcome = Literal["selected", "passed-over", "skipped"]
SkipReason = Literal["unknown", "unavailable", "unhealthy", "no-authored-tier"]  # the selector's own reasons


@dataclass(frozen=True)
class CandidateTrace:
    harness_id: str
    outcome: Outcome  # selected: the first member that passes; passed-over: a later member that would also pass
    reason: SkipReason | None  # set only when skipped
    model: ModelTrace | None
    effort: EffortTrace | None


@dataclass(frozen=True)
class SelectionTrace:
    basis: Literal["acceptable-set", "runner-default"]  # runner-default: the effective set was empty
    candidates: tuple[CandidateTrace, ...]  # every set member, in set order; exactly one outcome each
    selected: str | None  # None: every member skipped; the runner would escalate without spawning


class HarnessSelector:
    def select(self, node: EnvelopeNode) -> HarnessSelection: ...  # unchanged; built on the member check below

    def trace(self, model: list[str], harnesses: list[str]) -> SelectionTrace:
        # The per-member check is extracted from `select` and applied past the winner too.
        # Strict gate is `selection_is_strict`: model preference nonempty and (more than one member, or a
        # `blizzard:` tier in the preference). An empty set makes the runner's default harness the only candidate.
        ...
```

`Spawner` keeps calling `SessionResolver.session_stamps(node, resume, harness_id=...)`, which takes `.value` from the
traced results.

### Pools already running

```python
@dataclass(frozen=True)
class RecordedPool:
    harness: str
    model: str | None  # PoolHead.resolved_model; None means unknown
    effort: str | None  # PoolHead.resolved_effort
    differs: bool  # the recorded harness, model, or effort differs from the next fresh mint


# Read only through IReadLeaseSessionRepository.pool_head(chunk_id, session_name), and only when a chunk is named.
# The report opens no write path to the runner store.
```

## Open questions

- The names `Traced`, `SessionSources`, `ChunkDefaults`, `ModelTrace`, `EffortTrace`, `CandidateTrace`,
  `SelectionTrace`, `RecordedPool`, and `trace_model` are provisional; only their fields are settled. Engineering
  answers.
- The vocabulary of the reasons a `ModelTrace.tried` entry did not resolve is not settled. Engineering answers.
- `SessionResolver.pool_head_breach` rotates a named pool's head on `model-drift` when the node now resolves to a
  different model than the head recorded. That contradicts "a live pool keeps the model it was minted with on every
  resume", which the `recorded-differs` finding states. Product and the runner's owners answer which holds.
