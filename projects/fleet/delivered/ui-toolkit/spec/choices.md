# Choices

The kit's choice and navigation parts on Angular Aria, and the zone picker (`decision:ui-toolkit/angular-aria`).

## System

Only `fleet/src/lib/kit/` imports `@angular/aria`. A pattern whose container sits in an application template gets a kit
directive that carries the Aria directive through `hostDirectives`, forwarding its inputs. The template imports the
kit's directive. A directive on a kit component remains on its wrapper host; it does not move onto the component's
native button. The grid bridge below connects those two elements.

| Kit export                                                                  | Kind       | Carries                                                 | Placed on                                                                                                                     | Replaces                                |
| --------------------------------------------------------------------------- | ---------- | ------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- | --------------------------------------- |
| `KitTabs`                                                                   | component  | `ngTabs`, `ngTabList`, `ngTab`                          | —                                                                                                                             | hand-wired `role="tab"`                 |
| `fleetKitTabPanel`                                                          | directive  | `ngTabPanel`                                            | each panel a page projects into `KitTabs`                                                                                     | the page's own `@switch`                |
| `KitChips`                                                                  | component  | `ngListbox` (horizontal, single selection), `ngOption`  | —                                                                                                                             | `aria-pressed` toggles                  |
| `fleetKitChipGroup`                                                         | directive  | `ngListbox` (horizontal, single selection)              | the element holding a page's `@for` of `KitChip`s, such as the project lens, filter chips, project color, and graph list rows | one Tab stop per chip                   |
| `KitChip`                                                                   | component  | `ngOption` inside a `fleetKitChipGroup`                 | —                                                                                                                             | `[selected]` toggles                    |
| `fleetKitToolbar`, `fleetKitToolbarWidget`                                  | directives | `ngToolbar`, `ngToolbarWidget`                          | an action chip row and its chips (`[action]="true"` today)                                                                    | separate buttons                        |
| `fleetKitSelectList`                                                        | directive  | `ngListbox` (vertical, single selection)                | the element holding a page's `@for` of rows                                                                                   | a column of buttons                     |
| `KitSelectRow`                                                              | component  | `ngOption`                                              | —                                                                                                                             | `<button aria-current>`                 |
| `fleetKitAccordionGroup`                                                    | directive  | `ngAccordionGroup`                                      | the element holding a page's sections                                                                                         | —                                       |
| `KitAccordionSection`                                                       | component  | `ngAccordionTrigger`, `ngAccordionPanel`                | —                                                                                                                             | hand-wired `aria-expanded` and `region` |
| `fleetKitGrid`, `fleetKitGridRow`, `fleetKitGridCell`, `fleetKitGridWidget` | directives | `ngGrid`, `ngGridRow`, `ngGridCell`, `ngGridCellWidget` | `<table>`, `<tr>`, `<td>`, and the control in a cell                                                                          | per-button Tab stops                    |
| `KitCombobox`                                                               | component  | Angular Aria combobox over `ngListbox`                  | —                                                                                                                             | free-text zone                          |

Unchanged: `KitTab` and `KitTabStrip` used as routed main-nav links (`decision:ui-toolkit/nav-stays-links`); `KitMenu`
and `KitSelect`'s CDK keyboard models (`decision:ui-toolkit/cdk-parts-stay`); the graph-node, graph-session, and
proposal-count tables (`decision:ui-toolkit/no-table-engine`). The select gains the opt-in grid bridge below.

## Interfaces

```html
<!-- A page's master list: one Tab stop, Up and Down move, Enter opens. product:keyboard-choices -->
<div fleetKitSelectList [value]="selectedId()" (valueChange)="open($event)" ariaLabel="Cadences">
  @for (cadence of cadences(); track cadence.id) {
    <fleet-kit-select-row [value]="cadence.id">{{ cadence.name }}</fleet-kit-select-row>
  }
</div>

<!-- An interactive table: one Tab stop, arrow keys between cells, Enter or Space works the cell's control.
     Grid controls bind the wrapper's Aria widget to the actual native button. -->
<table fleetKitGrid ariaLabel="Members">
  @for (member of members(); track member.id) {
    <tr fleetKitGridRow>
      <td fleetKitGridCell>{{ member.username }}</td>
      <td fleetKitGridCell>
        <fleet-kit-select #roleSelect fleetKitGridWidget #roleWidget="fleetKitGridWidget"
          [gridFocusTarget]="roleSelect.controlElement()" gridWidgetType="complex"
          [gridDisabled]="roleSelect.disabled()" [controlTabIndex]="roleWidget.controlTabIndex()"
          (gridActivated)="roleSelect.openFromGrid()" (gridDeactivated)="roleSelect.closeFromGrid()"
          (opened)="roleWidget.activate()" (closed)="roleWidget.deactivate()" … />
      </td>
      <td fleetKitGridCell>
        <fleet-kit-button #remove fleetKitGridWidget #removeWidget="fleetKitGridWidget"
          [gridFocusTarget]="remove.controlElement()" gridWidgetType="simple"
          [gridDisabled]="remove.disabled()" [controlTabIndex]="removeWidget.controlTabIndex()"
          variant="danger">Remove</fleet-kit-button>
      </td>
    </tr>
  }
</table>

<!-- Tabs with their panels. -->
<fleet-kit-tabs [options]="tabs" [(active)]="tab">
  <section fleetKitTabPanel="findings">…</section>
  <section fleetKitTabPanel="proposals">…</section>
</fleet-kit-tabs>
```

| Grid bridge              | Contract                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `KitButton`, `KitSelect` | Expose `controlElement(): HTMLButtonElement \| undefined` from a view query; accept `controlTabIndex: number \| null`, bound to the native button's `tabindex`. `null` preserves standalone native tab order. Native `disabled` remains bound to the component's `disabled` input                                                                                                                                            |
| `fleetKitGridWidget`     | Hosts `GridCellWidget` on the wrapper, visible to `GridCell`'s content query; forwards public Aria inputs as `gridFocusTarget`, `gridWidgetType`, `gridDisabled`, and outputs as `gridActivated`, `gridDeactivated`; exports itself as `fleetKitGridWidget` and exposes public `activate()` and `deactivate()`                                                                                                               |
| DOM routing              | `fleetKitGridCell` and `fleetKitGridWidget` also set the `ngGridCell` and `ngGridCellWidget` host marker attributes, respectively, so Aria 22.2's ancestor lookup recognizes events originating at the nested native button                                                                                                                                                                                                  |
| Focus and Tab            | `gridFocusTarget` uses Aria 22.2's public `focusTarget`. The wrapper and a cell containing a widget stay at `tabindex="-1"`. `controlTabIndex()` is `0` only for the active cell's enabled control in an enabled grid, otherwise `-1`; it reads public cell/grid state, not Aria private patterns. The grid uses roving focus and skips disabled cells (`softDisabled=false`)                                                |
| Button activation        | A `simple` widget receives focus on its native button. Native Enter/Space generates the click exactly once; the adapter does not synthesize another click from `activated`. Disabled controls cannot activate                                                                                                                                                                                                                |
| Select activation        | A `complex` widget's Enter activation calls `openFromGrid()`; the bridge also handles Space by preventing its native default and calling Aria's public `activate()` once. `openFromGrid()` is idempotent and focuses the current CDK option after the overlay attaches. Opening emits `opened`, so pointer opening also activates the widget. The native trigger's click must not toggle it closed after keyboard activation |
| Popup ownership          | While the select is open, its CDK listbox owns arrows, Enter, and Space; grid navigation is suspended. Pick, Escape, or dismissal closes the popup, emits `closed`, deactivates the widget, and restores focus to the same native trigger. Aria deactivation calls idempotent `closeFromGrid()`; neither direction reopens the popup. Popup Escape does not also navigate the grid                                           |
| Outside a grid           | No widget directive or tabindex override is required; existing button and select keyboard behavior is unchanged                                                                                                                                                                                                                                                                                                              |

The grid-widget stories exercise two rows with enabled and disabled buttons and selects: one Tab entry, arrow movement,
the native focus target, exactly one activation, popup key ownership, and focus restoration after close.

```ts
// KitCombobox: a Signal Forms control (FormValueControl<string>). Pick only (product:picked-values).
export interface ComboOption {
  readonly value: string;
  readonly label: string;
  readonly meta?: string; // shown at the row's end, such as a UTC offset
}
// inputs: options: readonly ComboOption[]; ariaLabel; noneLabel?: string
// With noneLabel, an optional field lists a first option of that label whose value is '', so the field can be cleared.
// Typing filters options by case-insensitive substring of label. Up and Down move the highlight, Enter picks it,
// Escape closes. Leaving the input with text that picks nothing restores the text of the current value.
// A current value with no option is listed first as its own option, marked "not in list", and kept until another
// is picked.

// fleet/src/lib/core/zones.ts — decision:ui-toolkit/browser-zones
export function zoneOptions(now: number): readonly ComboOption[];
// Intl.supportedValuesOf('timeZone'), plus 'UTC' when absent, sorted by name. Each option's meta is its UTC offset
// at `now`, from Intl.DateTimeFormat(…, { timeZone, timeZoneName: 'shortOffset' }). Synchronous: no loading or
// failure state.
```

## Flows

| Part                | Enters                  | Moves                                          | Acts                                    |
| ------------------- | ----------------------- | ---------------------------------------------- | --------------------------------------- |
| Tabs                | Tab to the selected tab | Left and Right, Home, End                      | Selection follows focus                 |
| Single-choice chips | Tab to the chosen chip  | Left and Right                                 | Selection follows focus                 |
| Action chip row     | Tab to the first chip   | Left and Right                                 | Enter or Space runs the action          |
| Master list         | Tab to the active row   | Up and Down, Home, End, type-ahead             | Enter opens the row                     |
| Accordion           | Tab to a header         | Up and Down between headers                    | Enter or Space toggles                  |
| Combobox            | Tab to the input        | Typing filters; Up and Down move the highlight | Enter picks, Escape closes              |
| Interactive table   | Tab to the active cell  | Arrow keys between cells                       | Enter or Space works the cell's control |
