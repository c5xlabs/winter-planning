# UI toolkit decisions

## Signal Forms (decision:ui-toolkit/signal-forms)

Every board form is built on Angular Signal Forms (`@angular/forms/signals`). It is the one candidate shaped like the
forms the board already writes, a signal holding a draft. It adds what each form now builds alone: field state,
validation rules declared per model, and a submit that lands the hub's refusals on the fields they name. Reactive Forms
and template-driven forms are not used. The survey is in [candidates](research/landscape/candidates.md).

## Angular Aria (decision:ui-toolkit/angular-aria)

The kit's choice and navigation parts take their keyboard and ARIA behavior from Angular Aria (`@angular/aria`) where a
pattern fits: tabs, listbox, toolbar, accordion, combobox, and grid. It is first-party, unstyled, and built by the CDK's
own team, so it keeps the board's look. Only the kit imports it. Where a pattern's container sits in an application
template, as a list's rows, a table, or an accordion's sections do, the kit exports a directive that carries the Aria
directive as a host directive, and the template uses the kit's.

## Angular 22 first (decision:ui-toolkit/angular-22-first)

Both adoptions wait for the board's move to Angular 22, where Signal Forms and Angular Aria are stable, rather than
building on the experimental and developer-preview APIs that ship with 21. The upgrade is the epic's first landing. Its
toolchain is in [spec/platform.md](spec/platform.md).

## The frontend owns its domain (decision:ui-toolkit/frontend-domain)

The frontend models its own domain, as the backend does, and translates to and from the wire at one boundary. Forms,
views, and models work in frontend domain types. Only the data-access files and the translators see a generated wire
type, so a wire change is absorbed in one translator rather than in every component that read the field.

This reverses `bzh:frontend-wire-conformist`. That rule forbids the frontend from restating a wire shape, a vocabulary,
or a backend judgment, and it led every component to work in the generated types directly. In blizzard-context it is
replaced by `bzh:frontend-domain-boundary`, and every rule that cites the old one is reworded with it. The reversal is a
requirement of this epic (`product:domain-boundary`), not a convention change made beside it.

## Validation in both places (decision:ui-toolkit/validate-both)

A form validates as the operator types, against rules its domain model declares: required fields, value formats, and
any rule the frontend can judge from what it holds. The hub keeps validating every write as before. The two are allowed
to disagree, and the hub wins: a refusal the frontend did not foresee lands on its field like any other. The frontend's
rules are its own and are not generated from the wire's schema.

## Zones from the browser (decision:ui-toolkit/browser-zones)

The zone picker offers the browser's IANA zone list (`Intl.supportedValuesOf('timeZone')`), plus `UTC`, which some
browsers leave out. The hub's scheduler reads the same IANA database, so the two lists differ only by database version.
Where they do, the hub refuses the save and the refusal lands on the zone. No hub endpoint is added.

## A default is chosen, never inferred (decision:ui-toolkit/explicit-default)

When the wire reads a blank field as "the default", the domain model makes it a choice: the default, or a stated value.
The translator sends the wire's "default" only for that choice, never for an empty or malformed entry. Text that does
not read as the field's type is the field's own error, and nothing is sent.

## Forge is required in the form (decision:ui-toolkit/forge-required)

The repository and work-source forms require a forge, through their domain models' rules. The wire does not change: the
create requests keep `forge` optional in their schema, and the hub keeps refusing a create that names none. Making the
wire state it would be a breaking change under `bzh:released-surface-additive`, and it is outside this epic.

## No wire change (decision:ui-toolkit/no-wire-change)

The epic changes no route, request, response, or schema on the hub or the runner. Everything it needs from the wire
already exists, and everything it adds is frontend.

## The kit floor grows by two (decision:ui-toolkit/kit-dependencies)

`bzh:frontend-kit-floor` lets the kit depend on `@angular/core`, `@angular/cdk`, and the token sheet. The list grows to
include `@angular/aria` and `@angular/forms/signals`; the second is needed because `KitField` and the kit controls bind
a Signal Forms field directly. The kit still imports no query, mutation, service, or translator.

## Landing order (decision:ui-toolkit/landing-order)

1. Angular 22, with its toolchain.
2. The domain boundary: `bzh:frontend-domain-boundary` replaces `bzh:frontend-wire-conformist` in blizzard-context. Its
   sweep lands with an exemption list naming every unit that still reads a generated type outside the boundary.
3. The form base: `KitField`, `KitForm`, `KitNumberInput`, the controls' `[formField]` binding, and the shared
   `SaveOutcome` and refusal helpers.
4. The cadence form, end to end, as the exemplar: its domain model, translator, service, and form.
5. The admin config dialog, then the remaining forms.
6. The Aria parts: tabs, chip rows, and select lists, then accordions. Then `KitCombobox` on the cadence zone, then the
   three interactive tables.
7. The remaining read views, unit by unit, until the boundary's exemption list is empty.

Each landing removes the code it supersedes. The epic is delivered when the exemption list is empty.

## Main-nav tabs stay links (decision:ui-toolkit/nav-stays-links)

`KitTab` and `KitTabStrip` also draw the app's main navigation, where each tab is a routed link to a page. Navigation is
not the tabs pattern, so those uses stay links. Only `KitTabs`, which switches content within a page, moves to Angular
Aria.

## CDK menu and select stay where they are (decision:ui-toolkit/cdk-parts-stay)

`KitMenu` and `KitSelect` already have full keyboard models from the CDK menu and listbox, so they move to Angular Aria
only if the move buys something beyond consistency. Neither is migrated by default.

## No table engine (decision:ui-toolkit/no-table-engine)

No headless table library is adopted. None of the board's six tables sorts or filters, and nothing on the roadmap asks
them to. The three tables whose rows act take Angular Aria's grid for their keyboard model; the other three stay plain
`<table>`s.

## No styled component suite (decision:ui-toolkit/no-component-suite)

Angular Material, PrimeNG, Taiga UI, ng-zorro, and spartan/ui are not adopted. Each brings a visual language or a
styling system that would fight `tokens.css` on every surface, and replacing the kit wholesale is outside this epic.
