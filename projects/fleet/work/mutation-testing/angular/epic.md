---
epic: mutation-testing/angular
refinement: refined
status: planned
---

# Angular

Part of `initiative:mutation-testing`; it has no product requirements of its own yet.

## The problem

The board's code goes uncovered by mutation tooling. Its four projects run vitest through Angular's own
`@angular/build:unit-test` builder, and the one serious mutation tool for TypeScript cannot drive that builder, so a
weak frontend test is found only when a reviewer suspects it and mutates the line by hand.

## The outcome

Frontend mutation stays agent-driven through `ng test` under `bzh:mutation-review-selection` for now. Once the tooling
can drive the builder, the frontend adopts it with a `web:mutation` verification method of its own
(`decision:mutation-testing/angular/no-bypass`).
