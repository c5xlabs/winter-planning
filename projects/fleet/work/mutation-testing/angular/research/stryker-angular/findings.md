# StrykerJS and the Angular unit-test builder

As of August 2026 the tooling path is closed at the seam blizzard sits on. The frontend's four projects run vitest
through Angular's own `@angular/build:unit-test` builder, and [StrykerJS](https://stryker-mutator.io/) — the one serious
mutation tool for TypeScript — cannot drive it: its
[vitest runner](https://stryker-mutator.io/docs/stryker-js/vitest-runner/) needs to load and extend a vitest
configuration programmatically, an API the Angular builder does not expose. The gap is an open StrykerJS feature request
([stryker-js#5655](https://github.com/stryker-mutator/stryker-js/issues/5655), November 2025, no maintainer activity
yet), and Stryker's [official Angular guide](https://stryker-mutator.io/docs/stryker-js/guides/angular/) still assumes
Karma.

A bypass exists: a standalone per-project vitest config compiled through
[`@analogjs/vite-plugin-angular`](https://analogjs.org/docs/features/testing/vitest), the setup Stryker's vitest runner
is known to work with. Two of the vitest runner's standing limits would also apply when the seam opens: no browser mode
(immaterial — the real-Chromium `shell-sweep` specs are already excluded from `ng test`) and single-threaded runs per
Stryker worker.
