# The final-form gates

The two mechanical gates `final-form-rules` lands, and what each matches. Delivers `engineering:final-form-rules`. Paths
are under `blizzard/`, measured on `master` at `eda353d55`.

## System

Both gates are ast-grep rules, each named for the architecture rule it enforces: a rule in `contracts/ast-grep/rules/`,
a test in `contracts/ast-grep/rule-tests/` with `valid:` and `invalid:` snippets, a snapshot under
`rule-tests/__snapshots__/`, and an entry in `contracts/ast-grep/README.md`. `mise run structural-gate` runs
`ast-grep scan --error=unused-suppression` and then `ast-grep test`, in `.github/workflows/gate.yml` and
`mise run gate`. A suppression is `# ast-grep-ignore: <id>` on the line above the match, and a suppression that matches
nothing fails the gate.

## Interfaces

### A store write that refuses on a status-family fact

```yaml
# contracts/ast-grep/rules/store-refuses-stale.yml                          engineering:final-form-rules
id: bzh:store-refuses-stale
language: Python
severity: error
files: [src/blizzard/hub/store/**]
rule:
  pattern: $HELPER($$$)
  # $HELPER is one of the status-family helpers below.
  inside:
    stopBy: end
    any:
      - kind: with_statement            # a write the method opens
        has: { kind: with_clause, has: { pattern: $S.write($$$), stopBy: end } }
      - kind: function_definition       # a write the method joins
        has: { field: parameters, has: { regex: "transaction: (Locked)?Transaction", stopBy: end } }
```

The status-family helpers it matches, all in `src/blizzard/hub/store/internal/`:

| Module                                      | Helpers                                                                                   |
| ------------------------------------------- | ----------------------------------------------------------------------------------------- |
| `chunk_terminal_predicates.py`              | `chunk_is_terminal`, `maybe_live`, `settled_stopped`, `settled_done`, `done_completed_at` |
| `chunk_rows.py`                             | `is_terminal`, `currently_promoted_ats`                                                   |
| `chunk_decisions_store.py`                  | `not_closed_clause`, `decision_closure_ids`                                               |
| `garden_proposal_store.py`                  | `is_closed`                                                                               |
| `forge_record_store.py`, `secret_store.py`  | `referred_forge`, `standing_forge`, `secret_unavailable`                                  |
| `forge_referrers.py`, `secret_referrers.py` | `active_referrers`                                                                        |

It does not match a read: `list_open_questions` (`chunk_questions_store.py:41`) and `list_open_decisions`
(`chunk_decisions_store.py:193`) filter by `chunk_is_terminal` inside `read(...)`, and they pass. A write that
conditions on a status column inline, such as `WorkItemStore.edit`'s `closed_at IS NULL`, escapes the rule; review
catches it.

The helper list is the rule's whole reach, so a new status-family helper joins it in the same change that adds it.

### A broker call inside a store write

```yaml
# contracts/ast-grep/rules/publish-after-commit.yml                               engineering:final-form-rules
id: bzh:publish-after-commit
language: Python
severity: error
files: [src/blizzard/hub/store/**]
rule:
  any:
    - pattern: $X.publish($$$)
    - pattern: $X.publish_event_logged($$$)
    - kind: import_from_statement
      has: { kind: dotted_name, regex: "hub\\.events|hub_events|hub\\.domain\\.chunk\\.event_log", stopBy: end }
```

Nothing under `store/` matches today: the store's path to the broker went with `5ae6ebf28`.

## Flows

Each rule's test carries an `invalid:` snippet of the departure and a `valid:` snippet of the form it allows: for the
first, a write calling `is_terminal` and a read filtering by `chunk_is_terminal`; for the second, a store method
publishing and one returning the ids its caller publishes. Both scan the tree clean when they land, so every match the
first finds is moved or suppressed by then (the five judgment slices, [store-judgments.md](./store-judgments.md)).

## Open questions

- **The refuse-stale rule's reach.** Run against `src/blizzard/hub/store` on `eda353d55` (`ast-grep` 0.51.2, via
  `uvx --from ast-grep-cli`), the rule as written matches 4 sites: `record_promote`, `record_backlog_positions`, and
  `UrgencyChunksStore.move`, which are judgments, and `promoted_ats`, a read that takes `transaction` to join a unit, as
  every read a locked decision depends on does (write-entry.md). It misses most judgments: the fence's `is_terminal`
  (`chunk_rows.py:525`), the forge and secret checks in module-level helpers taking `conn` (`insert_record`,
  `update_record`, `_check_secret`, `_check_forge`), the retirements' referrer checks, and `GardenProposalStore`'s
  `self.is_closed(...)`, whose callee is an attribute the bare `$HELPER` pattern does not match. Matching every call
  form anywhere under `store/` finds 45 calls, most of them reads (fact loading, open-question and decision lists,
  counts). The scope "a write the method opens or joins" therefore neither holds every judgment nor excludes reads. A
  reshape to weigh: match any call form, inside a function that also inserts, updates, or deletes a row; and measure it
  again after the judgment slices, when it must scan clean. The publish rule matches nothing today, as expected.
