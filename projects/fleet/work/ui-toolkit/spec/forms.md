# Forms

How a form is built: a domain draft, its rules, a service that saves it, and the kit parts that render it. Built on
[the domain boundary](./boundary.md).

## System

| Part | Where | Holds |
| --- | --- | --- |
| `<concept>.domain.ts` | feature folder, or its kernel (`core/`) when a query or a second feature reads it | The draft type the form edits |
| `<concept>.form.ts` | feature folder | The draft's validation rules, as a Signal Forms schema |
| `<concept>.translate.ts` | beside its domain | Draft ⇄ wire, and the concept's `RefusalMap` |
| `<concept>.service.ts` | feature folder | `inject<Concept>Service()`: saves a draft, answers a `SaveOutcome` |
| Container | feature folder | Injects the service, owns the `form()`, submits it |
| Form view | feature folder | Presentational: takes the field tree, renders kit parts, emits `save` and `cancel` |
| `refusal.translate.ts` | `fleet/src/lib/core/forms/` | `readRefusal`: a hub refusal → domain fields, through a `RefusalMap` |
| `outcome.ts`, `submit.ts` | `fleet/src/lib/core/forms/` | `SaveOutcome`, `toFormErrors`, `submitForm` |
| `KitForm`, `KitField`, `KitNumberInput`, `KitOrDefault` | `fleet/src/lib/kit/` | Form chrome; bind Signal Forms fields (`decision:ui-toolkit/kit-dependencies`) |
| Kit controls | `KitTextInput`, `KitCheckbox`, `KitSelect`, `KitColorInput`, `KitCombobox` | Each implements Signal Forms' `FormValueControl`, so `[formField]` binds it; `valueChange` stays for use outside a form |

## Domain

```ts
// fleet/src/lib/core/forms/outcome.ts
export type SaveOutcome<D> =
  | { readonly ok: true }
  | { readonly ok: false; readonly fields: Partial<Record<keyof D, string>>; readonly form: string | null };

export interface RefusalMap<D> {
  /** A hub field path, normalized (see readRefusal), to the draft field it concerns; null when none does. */
  readonly field: (path: string) => keyof D | null;
  /** A problem `type` that concerns one field though it carries no path, such as a slug already taken. */
  readonly kinds: Readonly<Record<string, keyof D>>;
}
```

```ts
// garden/core/cadence.domain.ts — what the cadence form edits; no wire, no Angular
export type CadenceAnchor = 'scheduled' | 'started' | 'finished';
export interface CadenceDraft {
  readonly slug: string;
  readonly name: string;
  readonly cron: string;
  readonly zone: string;
  readonly spacing: string; // as typed: a whole number and m, h, or d
  readonly anchor: CadenceAnchor | 'unstated';
}

// admin/repositories/repository.domain.ts
export interface RepositoryDraft {
  readonly slug: string;
  readonly name: string;
  readonly forge: string; // a forge id; required here though optional on the wire (decision:ui-toolkit/forge-required)
  readonly owner: string;
  readonly repo: string;
  readonly baseBranch: string;
  readonly tokenSecret: string;
  readonly landingPolicy: LandingPolicy;
  readonly parkTimeoutDefault: boolean; // decision:ui-toolkit/explicit-default: the default is a choice
  readonly parkTimeoutHours: string; // as typed; judged only when the default is not chosen
}
```

```ts
// garden/cadences/cadence.form.ts — product:immediate-validation
const editRules = schema<CadenceDraft>((p) => {
  validate(p.cron, ({ value }) =>
    value() === '' || fiveFields(value())
      ? null
      : { kind: 'cron', message: 'Needs five fields: minute hour day month weekday' });
  validate(p.spacing, ({ value }) =>
    value() === '' || /^[1-9]\d*[mhd]$/.test(value())
      ? null
      : { kind: 'spacing', message: 'A whole number above zero and m, h, or d' });
  // The cross-field rules the hub states, judged here from the draft alone.
  validate(p.zone, ({ value, valueOf }) =>
    (value() === '') === (valueOf(p.cron) === '')
      ? null
      : { kind: 'schedule', message: 'A schedule needs both a cron and a zone' });
  validate(p.anchor, ({ value, valueOf }) =>
    value() !== 'unstated' && valueOf(p.spacing) === ''
      ? { kind: 'anchor', message: 'An anchor needs a spacing' }
      : null);
  validate(p, ({ value }) =>
    value().cron === '' && value().spacing === ''
      ? { kind: 'cadence', message: 'State a schedule, a spacing, or both' }
      : null);
});
export const cadenceEditRules = editRules; // the slug is fixed on edit
export const cadenceCreateRules = schema<CadenceDraft>((p) => {
  apply(p, editRules);
  required(p.slug);
});

// admin/repositories/repository.form.ts
export const repositoryRules = schema<RepositoryDraft>((p) => {
  for (const field of [p.slug, p.forge, p.owner, p.repo, p.baseBranch, p.tokenSecret]) required(field);
  disabled(p.parkTimeoutHours, ({ valueOf }) => valueOf(p.parkTimeoutDefault));
  validate(p.parkTimeoutHours, ({ value, valueOf }) =>
    valueOf(p.parkTimeoutDefault) || /^\d+$/.test(value())
      ? null
      : { kind: 'hours', message: 'Not a whole number of hours' });
});
```

The rules are the frontend's own (`decision:ui-toolkit/validate-both`). A rule the hub also enforces is written again
here on purpose. A rule the frontend cannot judge from what it holds, such as whether a slug is taken, is left to the
hub.

## Interfaces

```ts
// fleet/src/lib/core/forms/refusal.translate.ts — the one shared reader of the hub's refusal shapes.
// A loc is normalized before lookup: a leading "body" | "query" | "path" segment is dropped; string segments join
// with "."; a number segment becomes "[n]". So ["body","schedule","cron"], ["body","schedule.cron"], and
// ["body","chores",0,"key"] read as "schedule.cron", "schedule.cron", and "chores[0].key".
export function readRefusal<D>(error: unknown, map: RefusalMap<D>): SaveOutcome<D>;
//   a Problem whose detail is a ValidationEntry list -> each entry to map.field(path), else to `form`
//   a Problem whose type is in map.kinds             -> that field, with the problem's detail as its message
//   anything else (network failure, 5xx, unknown 4xx) -> `form`, worded by errorMessage as today

// garden/core/cadence.translate.ts
export const CADENCE_REFUSALS: RefusalMap<CadenceDraft> = {
  field: (path) =>
    ({
      slug: 'slug', name: 'name', 'schedule.cron': 'cron', 'schedule.zone': 'zone',
      schedule: 'cron', spacing: 'spacing', anchor: 'anchor',
    } as const)[path] ?? null,
  kinds: { 'urn:blizzard:problem:cadence-slug-taken': 'slug' }, // a 422 with a string detail, no path
};
// admin/repositories/repository.translate.ts: kinds holds 'urn:blizzard:problem:repository-slug-taken' (a 409)
// for slug, and 'urn:blizzard:problem:repository-coordinate-taken' for repo.

export function toCreateRequest(draft: CadenceDraft): CadenceCreateRequest;
// RepositoryDraft: parkTimeoutDefault -> park_timeout_hours: null; otherwise Number(parkTimeoutHours), reached only
// once the rule has passed.
```

```ts
// garden/cadences/cadence.service.ts — product:error-at-field
export function injectCadenceService(): {
  readonly pending: Signal<boolean>;
  create(draft: CadenceDraft): Promise<SaveOutcome<CadenceDraft>>;
  edit(id: string, before: CadenceDraft, after: CadenceDraft): Promise<SaveOutcome<CadenceDraft>>;
};
// Each call runs the existing mutation hook (injectCreateCadenceMutation, injectEditCadenceMutation) with the
// translated request, so its invalidations settle as bzh:frontend-mutation-settles-on-refresh requires, and maps a
// failure through readRefusal(error, CADENCE_REFUSALS). It never throws for a refusal.
```

```ts
// fleet/src/lib/core/forms/submit.ts
export function toFormErrors<D>(outcome: SaveOutcome<D>, form: FieldTree<D>): ValidationError[] | undefined;
//   each outcome.fields entry -> { kind: 'hub', fieldTree: form[key], message }; outcome.form -> the root field tree

/** Runs Signal Forms' submit: it marks every field touched and runs `save` only when the form is valid. Afterwards it
 *  focuses the first field in error, a rule's or the hub's, through that field's focusBoundControl(). Answers whether
 *  the save succeeded. */
export function submitForm<D>(form: FieldTree<D>, save: (draft: D) => Promise<SaveOutcome<D>>): Promise<boolean>;
```

```ts
// The container and its presentational form view (bzh:frontend-container-presentational)
// container: garden/cadences/gardening-cadences-page.ts
protected readonly cadences = injectCadenceService();
protected readonly draft = signal<CadenceDraft>(EMPTY_CADENCE_DRAFT);
protected readonly form = form(this.draft, cadenceCreateRules);
protected async onSave(): Promise<void> {
  if (await submitForm(this.form, (d) => this.cadences.create(d))) this.closeCreate();
}

// view: garden/cadences/cadence-form.ts — inputs and outputs only, no injection
readonly form = input.required<FieldTree<CadenceDraft>>();
readonly creating = input(false);
readonly save = output<void>();
readonly cancel = output<void>();
```

```html
<!-- cadence-form.html -->
<fleet-kit-form [form]="form()" (save)="save.emit()" (cancel)="cancel.emit()" [saveLabel]="creating() ? 'Create' : 'Save'">
  <fleet-kit-field label="Slug" [field]="form().slug">
    <fleet-kit-text-input [formField]="form().slug" />
  </fleet-kit-field>
  <fleet-kit-field label="Zone" [field]="form().zone">
    <fleet-kit-combobox [formField]="form().zone" [options]="zones" noneLabel="None" />
  </fleet-kit-field>
</fleet-kit-form>
```

| Kit part | Inputs | Shows |
| --- | --- | --- |
| `KitForm` | `form: FieldTree<unknown>`, `saveLabel`, `cancelLabel` | The field stack, the action row, and the root field's errors as the form-level line. Save is disabled while `form().submitting()`. Emits `save` on submit and `cancel` |
| `KitField` | `label`, `field: FieldTree<unknown>`, `hint?` | The label; a required marker when `field().required()`; the hint; `field().errors()` once `touched()` |
| `KitNumberInput` | `[formField]` bound to a `string`, `unit?` | A text input with `inputmode="numeric"` and the unit after it; the text is kept as typed, and the rule judges it |
| `KitOrDefault` | `[formField]` bound to the draft's `boolean` default choice, `defaultLabel` | A "default" chip beside the projected control, which the rules disable while the default is chosen |

Every kit control is focusable through `focusBoundControl()`, which `submitForm` relies on.

The admin config dialog stays generic over five record types. It takes the record type's field tree, and its field list
keeps only presentation (key, label, kind, order, and when a field is shown), keyed by the domain draft's fields; a
field's kind names the kit control it is drawn with. Required-ness and number checks move to each record type's
`*.form.ts`. The awaiting-human answers are one form per question, each over its own `AnswerDraft`;
the row's action chips are a toolbar of actions (`spec/choices.md`), not fields.

## Flows

```text
operator types / leaves a field
  -> the field's rules run; its errors show once it is touched

Save pressed  (KitForm emits save; the container calls submitForm)
  -> submit marks every field touched
       a rule fails       -> nothing is sent; focus moves to the first field in error
       every rule passes  -> service.create(draft)
            settles ok                  -> true; the container closes or resets the form
            refused                     -> readRefusal -> toFormErrors: each on its field, the rest on the form line
                                           -> focus the first field in error; the draft is untouched; false
            fails (network, 5xx, …)     -> the form line, worded as today; false
  -> a hub error on a field clears once the operator edits that field
```

The hub answers one refusal at a time for a cadence's schedule: `Schedule.parse` stops at the first bad field. A form
can therefore show one hub error where its own rules show several.
