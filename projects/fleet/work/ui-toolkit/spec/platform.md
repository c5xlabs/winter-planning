# Platform

## System

The web workspace (`web/package.json`) after the upgrade (`decision:ui-toolkit/angular-22-first`). Each row's range is
what the package declares for Angular 22.

| Package | Today | After | Supports Angular 22 by |
| --- | --- | --- | --- |
| `@angular/*` (core, common, compiler, forms, platform-browser, router, build, cli, compiler-cli, animations) | 21.2 | 22.2 | — |
| `@angular/cdk` | 21.2 | 22.2 | — |
| `ng-packagr` | 21.2 | 22.2 | peer `@angular/compiler-cli ^22` |
| `@angular/aria` | absent | 22.2 | peer `@angular/core ^22 \|\| ^23` |
| `typescript` | ~5.9 | ~6.0 | `@angular/compiler-cli` 22 peers `>=6.0 <6.1` |
| `@analogjs/vite-plugin-angular` | 2.8.1 | 2.9 | peer `@angular/build ^22.0.0` |
| `@storybook/angular-vite`, `storybook` | 10.6.2 | 10.6.7 | peer `@angular/core >=21 <23` |
| `angular-eslint` | 21.4.0 | 22.6 | `@angular-devkit/core >=22 <23` |
| `typescript-eslint` | 8.59 | 8.72 | peer `typescript <6.1.0` |
| `@tanstack/angular-query-experimental` | 5.101 | unchanged | peer `@angular/core >=16` |
| `@hey-api/openapi-ts` | 0.99 | unchanged | peer `typescript >=6.0.0` accepted |

```text
// The vite split and the js-yaml override recorded in web/package.json's "//" note carry over. The exact
// @angular/compiler-cli pin that @analogjs's optional peer chain holds moves with the Angular version, so the lock is
// resolved once under --legacy-peer-deps and re-proven under a strict `npm ci`, as that note directs.
```
