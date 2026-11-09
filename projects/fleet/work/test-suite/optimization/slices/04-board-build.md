---
name: board-build
---

# The board's build on every pull request

Builds both boards ahead of time on every pull request and publishes the build. Delivers `product:board-build-on-pr`;
builds `spec/pr-gate.md` §The board's build on every pull request. Independent of every other slice.

## Change

- `gate.yml`'s `frontend` job runs `npm run build` after `npm ci`.
- The job uploads `src/blizzard/static/` as `web-dist`, kept one day.

## Acceptance

- A pull request with a template error only the runner's board catches fails the `frontend` job.
- `web-dist` holds `src/blizzard/static/hub` and `src/blizzard/static/runner`.
