# Toolkit candidates

The component libraries and frontend accelerants the board does not yet use, judged against the gaps
[current.md](./current.md) measures: forms that each build their own field state, layout, and error handling; choice
parts with no keyboard model; free-text inputs where a picker belongs; and six small tables. The judgment criteria,
taken from what the board already is:

- **It keeps the look.** The board's visual language is `tokens.css`. A candidate that brings its own theme fights it on
  every surface.
- **It keeps the kit floor.** `bzh:frontend-kit-floor` lets the kit depend only on `@angular/core`, `@angular/cdk`, and
  the token sheet. A candidate the kit cannot wrap breaks that rule.
- **It keeps the wire the authority.** At the time of the survey, `bzh:frontend-wire-conformist` forbade the frontend
  from previewing a predicate the backend enforces on a write, so a forms library had to let validation come from the
  wire. `decision:ui-toolkit/frontend-domain` has since reversed that rule.
- **It fits the budget.** 600 kB initial bundle.

## Verdicts

| Candidate | What it is | Fit | Verdict |
| --- | --- | --- | --- |
| **Signal Forms** (`@angular/forms/signals`) | Angular's signal-native form model: a form is a signal of the model, every field has `value`, `touched`, `dirty`, `errors`, and a `submit()` that maps server errors back onto fields | Matches how every form already holds its draft. It replaces eleven hand-built drafts with one model, and its `submit()` lands the hub's refusals on their fields. Experimental in 21, **stable in 22** | **Adopt** (`decision:ui-toolkit/signal-forms`) |
| **Angular Aria** (`@angular/aria`) | First-party headless directives for the WAI-ARIA patterns: accordion, combobox, grid, listbox, menu, tabs, toolbar, tree. Behavior, focus, and keyboard only; no styles | Keeps the look, because it brings none. The kit can wrap it and stay first-party. It fills exactly the keyboard gaps in tabs, chips, select rows, and accordions, and adds the combobox the board lacks. Developer preview in 21, **stable in 22**, where the simple combobox was promoted to the primary `combobox` entry point | **Adopt** (`decision:ui-toolkit/angular-aria`) |
| CDK parts not yet used: table, scrolling, `LiveAnnouncer` | Already a dependency | Allowed by the kit floor with no decision. `LiveAnnouncer` answers the missing mutation announcement | Take as needed; no adoption decision |
| TanStack Table | Headless sort, filter, and paginate model | The six tables neither sort nor filter, and the roadmap asks for neither | **Not now** (`decision:ui-toolkit/no-table-engine`) |
| Angular Material | Google's Material Design suite on the CDK | Brings Material's visual language and theming system, which would fight `tokens.css` on every surface. Its form fields wrap Reactive Forms rather than Signal Forms | **No** (`decision:ui-toolkit/no-component-suite`) |
| PrimeNG, Taiga UI, ng-zorro | Full styled component suites | The same objection, with a heavier bundle and a third-party theme layer | **No** |
| spartan/ui | shadcn-style parts copied into the repo, on its own headless "brain" primitives | Copying in is attractive, but its styling assumes Tailwind, which the board does not use, and its primitives duplicate what Angular Aria now gives first-party | **No** |

## What the two adoptions answer

| Gap from [current.md](./current.md) | Answered by |
| --- | --- |
| Eleven hand-built drafts with no shared field state | Signal Forms' form model |
| About 66 re-typed label-over-control blocks | A kit field part bound to a Signal Forms field |
| `required: true` restating the wire | The `required` list read from the generated JSON Schema, for the marker only |
| A 422 flattened into one sentence under the form | Signal Forms `submit()` routing each `loc` to its field |
| Tabs, chips, select rows, accordions with no keyboard model | Angular Aria tabs, toolbar / listbox, listbox, accordion |
| Free-text zone, cron, and spacing | Angular Aria combobox, plus kit parts that read the value back in words |
| Interactive tables with a Tab stop per button | Angular Aria grid, on the three tables whose rows act |

## Status sources

Both libraries' stability was checked against the Angular 22 release coverage and the CDK 22.0.0 release notes, which
remove the developer-preview tag from `@angular/aria` and promote the simple combobox to the primary entry point. The
board is on 21.2, so both adoptions start with the Angular 22 upgrade (`decision:ui-toolkit/angular-22-first`).

- [Angular v22 released (InfoQ)](https://infoq.com/news/2026/08/angular-v22-released)
- [Angular 22 key features and changes (angular.love)](https://angular.love/angular-22-key-features-and-changes)
- [@angular/cdk 22.0.0 release notes](https://newreleases.io/project/yarn/@angular/cdk/release/22.0.0)
- [Angular Aria: a new standard for building accessible components (angular.love)](https://angular.love/angular-aria-a-new-standard-for-building-accessible-components)
- [Signal Forms: form submission (angular.dev)](https://angular.dev/guide/forms/signals/form-submission)
- [Signal Forms: validation (angular.dev)](https://angular.dev/guide/forms/signals/validation)
