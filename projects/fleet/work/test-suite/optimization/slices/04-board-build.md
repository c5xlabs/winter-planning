---
name: board-build
code:
  - blizzard/.github/workflows/gate.yml::frontend
---

# The board's build on every pull request

Builds both boards ahead of time on every pull request and publishes the build. Delivers `product:board-build-on-pr`;
builds `spec/pr-gate.md` §The board's build. Independent of every other slice.

## Change

- `gate.yml`'s `frontend` job runs `npm run build` in `web/` after `npm ci`.
- The job uploads `src/blizzard/static/` as `web-dist`, kept one day.
- `bundle-check` keeps its own build.

## Acceptance

- A pull request with a template error only the runner's board catches fails the `frontend` job.
- `web-dist` holds `src/blizzard/static/hub` and `src/blizzard/static/runner`.
