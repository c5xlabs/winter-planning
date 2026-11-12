# Choices

The kit's choice and navigation parts on Angular Aria, and the zone picker (`decision:ui-toolkit/angular-aria`).

## System

Only `fleet/src/lib/kit/` imports `@angular/aria`. A pattern whose container sits in an application template gets a
kit directive that carries the Aria directive through `hostDirectives`, forwarding its inputs. The template imports
the kit's directive.

| Kit export | Kind | Carries | Placed on | Replaces |
| --- | --- | --- | --- | --- |
| `KitTabs` | component | `ngTabs`, `ngTabList`, `ngTab` | — | hand-wired `role="tab"` |
| `fleetKitTabPanel` | directive | `ngTabPanel` | each panel a page projects into `KitTabs` | the page's own `@switch` |
| `KitChips` | component | `ngListbox` (horizontal, single selection), `ngOption` | — | `aria-pressed` toggles |
| `fleetKitToolbar`, `fleetKitToolbarWidget` | directives | `ngToolbar`, `ngToolbarWidget` | an action chip row and its chips (`[action]="true"` today) | separate buttons |
| `fleetKitSelectList` | directive | `ngListbox` (vertical, single selection) | the element holding a page's `@for` of rows | a column of buttons |
| `KitSelectRow` | component | `ngOption` | — | `<button aria-current>` |
| `fleetKitAccordionGroup` | directive | `ngAccordionGroup` | the element holding a page's sections | — |
| `KitAccordionSection` | component | `ngAccordionTrigger`, `ngAccordionPanel` | — | hand-wired `aria-expanded` and `region` |
| `fleetKitGrid`, `fleetKitGridRow`, `fleetKitGridCell`, `fleetKitGridWidget` | directives | `ngGrid`, `ngGridRow`, `ngGridCell`, `ngGridCellWidget` | `<table>`, `<tr>`, `<td>`, and the control in a cell | per-button Tab stops |
| `KitCombobox` | component | Angular Aria combobox over `ngListbox` | — | free-text zone |

Unchanged: `KitTab` and `KitTabStrip` used as routed main-nav links (`decision:ui-toolkit/nav-stays-links`); `KitMenu`
and `KitSelect` (`decision:ui-toolkit/cdk-parts-stay`); the graph-node, graph-session, and proposal-count tables
(`decision:ui-toolkit/no-table-engine`).

## Interfaces

```html
<!-- A page's master list: one Tab stop, Up and Down move, Enter opens. product:keyboard-choices -->
<div fleetKitSelectList [value]="selectedId()" (valueChange)="open($event)" ariaLabel="Cadences">
  @for (cadence of cadences(); track cadence.id) {
    <fleet-kit-select-row [value]="cadence.id">{{ cadence.name }}</fleet-kit-select-row>
  }
</div>

<!-- An interactive table: one Tab stop, arrow keys between cells, Enter or Space works the cell's control. -->
<table fleetKitGrid ariaLabel="Members">
  @for (member of members(); track member.id) {
    <tr fleetKitGridRow>
      <td fleetKitGridCell>{{ member.username }}</td>
      <td fleetKitGridCell><fleet-kit-select fleetKitGridWidget … /></td>
      <td fleetKitGridCell><fleet-kit-button fleetKitGridWidget variant="danger">Remove</fleet-kit-button></td>
    </tr>
  }
</table>

<!-- Tabs with their panels. -->
<fleet-kit-tabs [options]="tabs" [(active)]="tab">
  <section fleetKitTabPanel="findings">…</section>
  <section fleetKitTabPanel="proposals">…</section>
</fleet-kit-tabs>
```

```ts
// KitCombobox: a Signal Forms control (FormValueControl<string>). Pick only (product:picked-values).
export interface ComboOption {
  readonly value: string;
  readonly label: string;
  readonly meta?: string; // shown at the row's end, such as a UTC offset
}
// inputs: options: readonly ComboOption[]; ariaLabel
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

| Part | Enters | Moves | Acts |
| --- | --- | --- | --- |
| Tabs | Tab to the selected tab | Left and Right, Home, End | Selection follows focus |
| Single-choice chips | Tab to the chosen chip | Left and Right | Selection follows focus |
| Action chip row | Tab to the first chip | Left and Right | Enter or Space runs the action |
| Master list | Tab to the active row | Up and Down, Home, End, type-ahead | Enter opens the row |
| Accordion | Tab to a header | Up and Down between headers | Enter or Space toggles |
| Combobox | Tab to the input | Typing filters; Up and Down move the highlight | Enter picks, Escape closes |
| Interactive table | Tab to the active cell | Arrow keys between cells | Enter or Space works the cell's control |
