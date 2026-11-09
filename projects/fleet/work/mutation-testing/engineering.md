# Mutation testing engineering

The engineering requirements of `initiative:mutation-testing`.

## What already holds

- The Python suite's unit and component tiers (`uv run pytest`, xdist-parallelized) re-run cheaply. The service, e2e,
  crash, and journey tiers spawn real daemons and fixture workspaces, and re-running those per mutant, thousands of
  mutants at minutes apiece, is not a cost any cadence absorbs.
- The backend corpus is roughly 48k lines under `src/blizzard/`.
- The frontend is four Angular projects, about 120 spec files, running vitest through the Angular CLI's
  `@angular/build:unit-test` builder.

## Brief requirements

- engineering:fast-tiers-only — Only the unit and component tiers run under a mutation tool.
- engineering:matrix-row — Every mutation method lands as a verifiability-matrix row with a stable method id
  (`bzh:verifiability-matrix`).
