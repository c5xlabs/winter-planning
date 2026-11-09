# Angular decisions

## No bypass for Stryker (decision:mutation-testing/angular/no-bypass)

The standalone per-project vitest config through `@analogjs/vite-plugin-angular` is not built, because it is a second
test configuration drifting beside `angular.json`, maintained for the mutation tool alone. Stryker is adopted with a
`web:mutation` matrix row when stryker-js#5655 closes or Angular's builder grows the programmatic seam.
