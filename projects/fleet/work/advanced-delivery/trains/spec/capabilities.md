# Capabilities, narrowed to the repository

Paths are relative to `blizzard/src/blizzard/`.

## Domain

```python
# hub/domain/config/capabilities.py — product:probed-capabilities
class Capability(StrEnum):
    ...  # every kind capability, unchanged
    CHANGES_CLOSE = "changes.close"  # kind capability
    QUEUE_JOIN = "queue.join"  # probed

# Kind capabilities: offered by every repository on a forge of that kind, as before.
# Probed capabilities: a kind may offer them; each repository must be shown to have them.
PROBED_CAPABILITIES: frozenset[Capability] = frozenset({Capability.QUEUE_JOIN, Capability.REFS_ADVANCE})
```

```python
# A repository's last probe.
@domain_model
@dataclass(frozen=True)
class Probes:
    shown: frozenset[Capability]  # the probed capabilities the probe showed
    protected: bool  # `protected` on GET /repos/{o}/{r}/branches/{base}
    taken_at: datetime
    # Set when the forge refuses an advance as protected; leaves refs.advance out of `shown` through every scheduled
    # probe until one taken on demand clears it (decision:advanced-delivery/trains/remembered-refused-advance).
    advance_refused_at: datetime | None
```

```python
# hub/forge/registry.py — a probe is the registry's, not a role a caller is handed; each kind's adapter supplies
# the reads below and the registry calls them.
class ForgeRegistry:
    async def probe(self, repository: RepositoryId) -> Probes: ...

    def offered(self, forge: ConfiguredForge, repository: RepositoryId | None = None) -> frozenset[Capability]:
        """The kind's capabilities, narrowed by the repository's last probe — still the one place a caller asks.
        A probe showing queue.join also leaves out changes.merge.merge, .squash, and .rebase: a merge queue on
        the base refuses a direct merge."""
```

| Capability     | GitHub read                                                                                                                                                                                                                                 | GitLab read                                                         |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| `queue.join`   | GraphQL `repository.mergeQueue(branch: <base>)` answers a queue (found whether a ruleset or classic protection requires it)                                                                                                                 | `merge_trains_enabled` and `merge_pipelines_enabled` on the project |
| `refs.advance` | No `pull_request`, `merge_queue`, `required_linear_history`, `required_signatures`, or `update` rule among the base's active rules (`GET /repos/{o}/{r}/rules/branches/{base}`), and no `advance_refused_at` since the last on-demand probe | Never offered by the kind                                           |

Classic branch protection is unreadable in detail without admin access; whether a branch is protected at all is
readable, and the probe records it.

## Interfaces

### When a probe is taken

| Trigger                                                              | On failure (5xx, timeout, rate limit)             |
| -------------------------------------------------------------------- | ------------------------------------------------- |
| The repository is created                                            | The last answer stands; retried at the next check |
| Its policy, its batching, or its forge changes                       | The last answer stands; retried at the next check |
| A visit's capability check finds the cached answer older than 10 min | The last answer stands; retried at the next check |
| First boot after the migration, every repository                     | Retried at the next visit's check                 |
| `blizzard hub repo probe <repository>`, on demand                    | Clears `advance_refused_at`                       |

```python
# A visit's capability check on a repository never yet probed, needing a probed capability:
answer = "pending"  # until a probe succeeds; never a refusal because the forge blinked
# A repository whose policy needs a probed capability it lacks (a fast-forward base only pull requests may change):
event("forge-capability-missing"); escalate(EscalationCause.FORGE_CAPABILITY_MISSING)  # at its next visit
# A merge-on-checks repository whose probe now shows queue.join:
escalate(EscalationCause.FORGE_CAPABILITY_MISSING)  # names the capability, points at forge-queue
```

`GET /api/repositories/{repository}` carries the last probe, when it was taken, and `advance_refused_at`
([surfaces.md](./surfaces.md#repositories-api)).

### New seam operation

`create_ref` and `delete_ref` are the policies epic's, on `IForgeRefs`.

```python
# hub/forge/landing.py
OPERATION_CAPABILITY["close_change"] = Capability.CHANGES_CLOSE


class IForgeChanges(Protocol):
    ...
    async def close_change(self, number: int, comment: str) -> None:
        """GitHub: PATCH /repos/{o}/{r}/pulls/{n} to closed, and a comment. Async, through the forges epic's
        retry policy."""
```

| Kind   | Offers `changes.close` | Can host the flow's own train                                                                                     |
| ------ | ---------------------- | ----------------------------------------------------------------------------------------------------------------- |
| GitHub | Yes                    | Yes                                                                                                               |
| GitLab | Yes                    | No: offers neither `branches.merge` nor `refs.advance`; its line goes to its merge train ([queue.md](./queue.md)) |

## The batching settings

```python
# hub/domain/config/repositories.py — RepositoryFields and RepositoryEdit gain each; _FIELDS and _DIFF_FIELDS too.
batch_size: int = 1  # 1..20; 1 lands each submission alone, as the flows epic does
batch_wait_seconds: int = 60  # how long the oldest ready submission waits for others
train_prefix: str = "blizzard-train/"  # branches trains are proven under
staging_prefix: str = "blizzard-staging/"  # branches trains are assembled under
```

| Rule                                                                                                                                                                                 | Refusal                                        |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------- |
| `batch_size` above 1 only beside `fast-forward` or `merge-on-checks`                                                                                                                 | `422 batching_not_applicable`                  |
| `batch_size` above 1 needs `refs.create`, `refs.delete`, `branches.merge`, `refs.advance`, `checks.read`, and for `merge-on-checks` `changes.close`, in `offered(forge, repository)` | `422 forge_capability_missing`, as a policy is |
| `train_prefix`, `staging_prefix`: valid branch prefixes, neither the base branch nor a prefix of it, neither a prefix of the other                                                   | `422`                                          |

A probe leaves `refs.advance` out wherever the base requires linear history or signed commits, so such a repository
cannot batch: a train lands merge commits whatever convention the repository keeps.

The capabilities are checked again as each batch forms. A flow whose repository has lost one forms no batch; its ready
members land alone, with a `forge-capability-fallback` event naming the capability.

## Open questions

- Where a repository's last probe and its `advance_refused_at` are stored: no table or column is named. Engineering.
- The field names and types of `Probes`; the spec names only its contents and `advance_refused_at`. Engineering.
- The code (`422`) and error class for an invalid `train_prefix` or `staging_prefix`. Engineering.
- What `close_change` answers, and its GitLab request. Engineering.
- The ten-minute probe age and the probed set (`PROBED_CAPABILITIES` above) have no names in the spec; both are
  placeholders. Engineering.
