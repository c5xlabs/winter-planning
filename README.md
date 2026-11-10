# ❄️ Winter requirements

Winter is the suite by [C5X Labs](https://github.com/c5xlabs) for building software with coding agents: **Winter
Workspace**, **Winter Fleet**, and the platform that fronts them. This repository holds the suite's requirements content
only — charters, milestones, epics, and the requirements and specs behind them. Code and its documentation live in each
product's own repositories.

## Contents

The repository is organized by project, then by initiative and epic: one folder per project under `projects/`, and
inside each, one folder per initiative or standalone epic, with an initiative's epics inside its folder.

| Path                                                  | What it holds                                                                                                          |
| ----------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `charter/`                                            | The suite's mission and the personas more than one product serves                                                      |
| `projects/platform/`                                  | The platform: the `winter` launcher, distribution, and the extension contract                                          |
| `projects/workspace/`                                 | Winter Workspace: repositories, feature environments, and services for agents                                          |
| `projects/fleet/`                                     | Winter Fleet: a hub and its runners, running coding agents unattended                                                  |
| `projects/<project>/charter/`                         | The project's mission, and the personas only that project serves                                                       |
| `projects/<project>/milestones.md`                    | The project's unreached milestones, in the order they are meant to be reached, then the ideas no milestone needs yet   |
| `projects/<project>/delivered.md`                     | The project's reached milestones, newest first                                                                         |
| `projects/<project>/decisions.md`                     | Decisions that span the project's epics or that no epic owns                                                           |
| `projects/<project>/work/`                            | Live epics and initiatives, one folder each                                                                            |
| `projects/<project>/delivered/`                       | Delivered epics, frozen as they shipped                                                                                |
| `projects/<project>/research/`                        | Findings that stop short of a choice, one folder per subject                                                           |
| `projects/<project>/work/<initiative>/initiative.md`  | An initiative's intent                                                                                                 |
| `projects/<project>/work/<initiative>/<epic>/epic.md` | An epic's intent: its problem and outcome                                                                              |
| `projects/<project>/work/<initiative>/<epic>/…`       | `requirements.md`, `mockups/`, `engineering.md`, `assumptions.md`, `spec.md` or `spec/`, `slices/`, and `decisions.md` |

A standalone epic sits directly under `work/` without an initiative folder. A delivered epic moves to the same path
under its project's `delivered/` and is not edited afterwards. Epics migrated from earlier planning may still carry part
plans, `spec/overview.md`, or `spec/carry-over.md` files from their original layout.

## Upstream schemes

How a reader resolves the upstream ids that requirements and assumptions cite.

| Scheme     | Resolves to                                                                        | Example                                |
| ---------- | ---------------------------------------------------------------------------------- | -------------------------------------- |
| `product:` | This repository: an epic's `requirements.md`, or the requirements in its `epic.md` | `product:fleet/chat/answer-in-one-tap` |
| `assume:`  | `assumptions.md` in the folder citing it, or the nearest one outward               | `assume:fleet/chat/one-bot`            |

## Frontmatter

Every `epic.md`, `initiative.md`, requirement leaf, and slice opens with YAML frontmatter.

```yaml
# epic.md
---
epic: adapters/codex
refinement: refined
status: planned
external: github:c5xlabs/winter-planning#12
---
```

```yaml
# initiative.md
---
initiative: adapters
refinement: refined
external: github:c5xlabs/winter-planning#3
epics:
  - claude-code
  - codex
---
```

```yaml
# requirements/<subject>.md
---
epic: adapters/codex
---
```

| Key          | Meaning                                          | Values                                                                                                       |
| ------------ | ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------ |
| `epic`       | The epic's id, without `epic:` or the project    | `<epic>` or `<initiative>/<epic>`                                                                            |
| `initiative` | The initiative's id                              | Its slug, matching its folder                                                                                |
| `refinement` | How far the intent has been worked out           | `outline` (a sketch), `refined` (ready to derive from), `final` (the product owner has declared it complete) |
| `status`     | Where the epic is                                | `planned` (not started), `in-progress` (work has started), `delivered` (it reached its users)                |
| `external`   | The item standing for the epic in another system | One `<system>:<id>`                                                                                          |
| `epics`      | The epics an initiative holds                    | Their slugs                                                                                                  |

Files are Markdown, formatted to a 120-column width.

A slice's keys:

```yaml
---
name: hold-store              # matches the filename after its build-order number
external: github:c5xlabs/winter-workspace#40
depends-on:                   # a slice of this epic by name, or an epic or initiative by id
  - schema
code:                         # where the behaviour it changes lives today
  - winter-workspace/src/env/reset.rs::reset
---
```

## Markers

- `planned`, `in-progress`, `delivered` — a milestone chart's status for each epic, matching the epic's own `status:`.
- **Proposed.** — a requirement not yet agreed.
- **Open.** — a decision not yet made; **Decider:** names who makes it.

## Ids

| Prefix         | Names                                       | Defined in                                                                      |
| -------------- | ------------------------------------------- | ------------------------------------------------------------------------------- |
| `persona:`     | A person the suite serves                   | A card in `charter/personas/` or `projects/<project>/charter/personas/`         |
| `initiative:`  | A durable investment broader than one epic  | The initiative's `initiative.md`                                                |
| `epic:`        | A major capability a product must establish | The epic's `epic.md`, under `work/` or `delivered/`                             |
| `product:`     | A requirement of an epic or of its owner    | A heading in the owner's `requirements.md`, or inline in `epic.md`              |
| `milestone:`   | A destination visible in a person's work    | A section in a `milestones.md`, then in the `delivered.md` beside it            |
| `decision:`    | A decision, open or made                    | A heading in the owner's `decisions.md`                                         |
| `engineering:` | What engineering commits to                 | A heading in the owning folder's `engineering.md`, or a project's or the root's |
| `assume:`      | Intent engineering assumed, unstated        | A heading in the `assumptions.md` beside the requirements resting on it         |

An id's first segment is its project: `epic:fleet/chat`, `persona:fleet/product-owner`. Inside a project an id may drop
that segment, and resolves in the project first, then at the program. An epic under an initiative carries it,
`epic:fleet/adapters/codex`. A bare `product:`, `engineering:`, or `assume:` id resolves from the citing folder outward
to the root; written in full, it carries its owner's epic id, `engineering:fleet/chat/one-delivery-per-answer`. Ids are
cited as plain text; find one by searching for the heading or file carrying it. Browse the live work under each
project's `work/`, such as [`projects/fleet/work/`](./projects/fleet/work/).

## License

Source-available under the Functional Source License 1.1 (FSL-1.1-ALv2). Each release converts to Apache License 2.0 two
years after it ships. See LICENSE.
