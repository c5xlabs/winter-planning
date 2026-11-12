# The board's UI landscape today

What the web board builds its surfaces from, measured on blizzard `master` at `eda353d55`, before this epic changes any
of it. Parent: [candidates.md](./candidates.md) judges what could fill the gaps named here; the
[spec](../../spec/) is built against this baseline.

## The stack

| Layer         | What                                                                                  |
| ------------- | ------------------------------------------------------------------------------------- |
| Framework     | Angular 21.2, zoneless, signals throughout, `OnPush` everywhere                       |
| Primitives    | `@angular/cdk` 21.2: overlay, listbox, menu, a11y, portal, drag-drop                  |
| Data          | TanStack Query (`@tanstack/angular-query-experimental`) over the generated hey-api client |
| Visualization | vis-timeline (tracks), dagre (graphs)                                                 |
| Design layer  | `fleet/src/lib/core/design/tokens.css`: one token sheet, monospace, dark mission-control |
| Workshop      | Storybook 10 over the kit, from `epic:storybook`                                      |
| Budget        | 600 kB initial-bundle warning, 700 kB error; 4 kB per component style                 |

`@angular/forms` is a dependency, but nothing in the hub, the runner, or `fleet` imports it.

## The kit already exists

The epic's premise, that every surface is invented by hand, no longer holds at the level of chrome. `fleet/src/lib/kit/`
holds some forty presentational parts, each with stories, and `bzh:frontend-kit-floor` makes using them a gate: ESLint
refuses a raw `<button>`, `<select>`, or `<input>` outside the kit, and the structural gate refuses re-typed panel
chrome.

| Part family    | Parts                                                                 | Uses across the apps |
| -------------- | --------------------------------------------------------------------- | -------------------: |
| Surface        | panel, dialog, confirm dialog, master-detail, accordion section       |                 ~110 |
| State          | async state (loading / error / empty), skeleton, error line           |                  199 |
| Action         | button, close button, menu, menu item, menu item radio, link tile     |                 ~130 |
| Choice         | select, select row, chips, chip, option, tabs, tab, checkbox          |                  ~75 |
| Entry          | text input (single or multiline), color input                         |                   23 |
| Display        | badge, beacon, fact list, prose block, markdown, glyph, swatch, gauges |                 ~160 |

What the kit stands on:

| Part                    | Built on                                    | Keyboard                                          |
| ----------------------- | ------------------------------------------- | ------------------------------------------------- |
| `KitSelect`             | CDK listbox + overlay, 170 lines            | Arrow keys and type-ahead, from the CDK listbox   |
| `KitMenu`, `KitMenuItem` | CDK menu + overlay                          | Full menu keyboard, from the CDK                  |
| `KitDialog`             | CDK a11y focus trap                         | Focus trapped, Escape closes                      |
| `KitTabs`, `KitTab`     | Hand-wired `role="tablist"` / `role="tab"`  | **None**: each tab is a Tab stop, no arrow keys   |
| `KitChips`              | Hand-wired toggle buttons, `aria-pressed`   | **None**: a single choice reads as N toggles      |
| `KitSelectRow`          | One `<button>` per row, `aria-current`      | **None**: a list of 40 rows is 40 Tab stops       |
| `KitAccordionSection`   | Hand-wired `aria-expanded` / `region`       | Enter and Space only; no arrow keys between headers |
| `KitTextInput`          | Native `<input>` / `<textarea>`             | Native                                            |

## Where the hand-building still is

### Forms

Eleven surfaces take typed input: the generic admin config dialog, which serves projects, repositories, forges,
secrets, and work sources, plus the project-branch dialog, the cadence and chore forms, and the garden's finding-triage,
proposal pass and accept, run, routine, and scope surfaces, along with the chunk's awaiting-human answer.

Each one builds the same things again:

- **Draft state.** Each form keeps its own draft in a `linkedSignal` or a signal map, with a `set(field, value)` helper
  and a `save()` that emits the whole draft. There is no shared notion of a touched, dirty, or invalid field.
- **Field layout.** Each form re-types a label-over-control block under its own class names: `cf-field` / `cf-label`
  in the cadence form, `chf-field` / `chf-label` in the chore form, `field` / `label` in the config dialog, and about
  66 such blocks across the eleven surfaces. They agree only because they were copied.
- **Required fields.** The config dialog's field lists (`projects.model.ts`, `repositories.model.ts`, …) hand-write
  `required: true` for fields the wire already marks required in its OpenAPI schema, so the frontend carries a second
  copy of what the backend requires. The copy has already drifted. `forge` is required in `REPOSITORY_FIELDS` and in
  the work-source fields, but optional in both `RepositoryCreateRequest` and `WorkSourceCreateRequest`, while the hub
  refuses a create naming no forge regardless: the frontend is right and the schema understates the hub.
- **Errors.** A refused save renders as one `KitErrorLine` under the whole form. The hub answers a malformed write with
  a 422 whose `detail` is a list of entries, each naming the field it refuses by `loc`. `errorMessage` in
  `fleet/src/lib/core/error-message.ts` flattens that list into one sentence
  (`schedule.cron: invalid cron; slug: taken`), so the operator reads which field is wrong and then finds it themselves.
- **Client checks.** The only client-side checks are the config dialog's `missingRequired` and `invalidNumbers`.
  Everything else is judged by the hub on save, which `bzh:frontend-wire-conformist` required at the time of this
  survey: a check that previews a predicate the backend enforces on a write was a backend judgment that belonged on
  the wire.

### The wire reaches every layer

`bzh:frontend-wire-conformist` asks the frontend to take every shape and vocabulary from the generated client and to
model nothing of its own, so components, view models, and forms all work in the generated types. Counted over the hub,
the runner, and `fleet` (specs, stories, and the generated client itself excluded), 255 files import a generated wire
type:

| Files                                        | Count |
| -------------------------------------------- | ----: |
| Data access (`*.query.ts`, `*.mutations.ts`) |    49 |
| View models (`*.model.ts`)                   |    60 |
| Components with a template                   |    86 |
| Other (helpers, shared logic)                |    60 |

The 206 outside data access are where a wire change lands today.

### Free text where a picker belongs

The cadence form takes its zone as free text with `America/Chicago` as the placeholder, its cron as free text, and its
spacing as free text with `10d` as the placeholder. A typo is found only when the hub refuses the save.

### Tables

Six hand-written `<table>`s, about 400 lines of CSS between them:

| Table                     | Rows act?                   |
| ------------------------- | --------------------------- |
| Members                   | Yes: change role, remove    |
| Cadences (cadence panel)  | Yes: edit, retire, run      |
| Routines (routine panel)  | Yes: edit, run              |
| Graph nodes               | No                          |
| Graph sessions            | No                          |
| Routine proposal counts   | No                          |

None of them sort, filter, or virtualize, and nothing on the roadmap asks them to.

### Smaller gaps

- Raw buttons, each under an element-level suppression, remain in seven files: the chunk card, the chunk list row,
  the chunk timeline, the issue list, the lens picker, the step drawer, and the delivery chunk.
- Drag-to-reorder uses CDK drag-drop directly in four places: board columns, the board list, and project links.
- Nothing announces a mutation's outcome to a screen reader, and nothing shows a transient confirmation.
