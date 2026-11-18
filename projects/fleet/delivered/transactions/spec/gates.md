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
  kind: call
  has: { field: function, regex: "(^|\\.)($HELPERS)$" }   # any call form: bare, self., or module.
  inside:
    kind: function_definition
    stopBy: end
    any:
      - has: { field: name, regex: "^(fence|mint_admission|_check_\\w+)$" }   # a write's guard helper
      - has:                                                                 # a function that writes a row
          stopBy: end
          kind: call
          has:
            field: function
            kind: attribute
            all:
              - has: { field: attribute, regex: "^(insert|update|delete)$" }
              - has: { field: object, regex: "(_store|stmts|statements_of\\(.*\\)|^sa|^s)$" }
```

`$HELPERS` is the status-family helpers below, all in `src/blizzard/hub/store/internal/`:

| Module                                      | Helpers                                                                                   |
| ------------------------------------------- | ----------------------------------------------------------------------------------------- |
| `chunk_terminal_predicates.py`              | `chunk_is_terminal`, `maybe_live`, `settled_stopped`, `settled_done`, `done_completed_at` |
| `chunk_rows.py`                             | `is_terminal`, `currently_promoted_ats`                                                   |
| `chunk_decisions_store.py`                  | `not_closed_clause`, `decision_closure_ids`                                               |
| `garden_proposal_store.py`                  | `is_closed`                                                                               |
| `forge_record_store.py`, `secret_store.py`  | `referred_forge`, `standing_forge`, `secret_unavailable`                                  |
| `forge_referrers.py`, `secret_referrers.py` | `active_referrers`                                                                        |
| `flow_store.py`                             | `has_open_turn`, `_busy`, `_ahead_on`                                                     |

A write is a function that builds an insert, update, or delete statement, or one of the guard helpers a write calls
before it writes (`fence`, `mint_admission`, `_check_secret`, `_check_forge`); a read builds none, so the rule passes
it. The statement-builder receivers are named so that a `dict.update` does not count as a write.

Measured on `eda353d55` (`ast-grep` 0.51.2, via `uvx --from ast-grep-cli`), the rule matches 19 calls, every one a
judgment the five judgment slices remove, and no read:

| Family             | Matches                                                                                                                            |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| Chunks             | `fence`, `mint_admission` (`chunk_rows.py:525,544`); `record_promote`, `record_backlog_positions`                                  |
| Forges and secrets | `_check_secret` (twice), `_check_forge`, `insert_record`, `update_record`, both `record_lifecycle_fact`s, `record_forge_lifecycle` |
| Garden and urgency | `GardenProposalStore.edit`, `attach`, `detach`; `UrgencyChunksStore.move`                                                          |
| Flows              | `FlowStore.take_turn`'s `has_open_turn`, `_busy`, `_ahead_on`                                                                      |

It does not reach a judgment made inline or one level down: `WorkItemStore.edit`'s `closed_at IS NULL`,
`TenantAdministrationStore.rename`'s `~_DELETING`, `TenantTeardownStore.close`'s last-tenant count, `FlowStore.close`'s
`has_open_turn` (its write is in `close_submission`), and `withdraw_waiting`'s choice in `flow_rows.py`. Review catches
those, against `bzh:store-refuses-stale`. The helper list and the guard-helper names are the rule's whole reach, so a
new status-family helper or guard joins it in the same change that adds it.

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
