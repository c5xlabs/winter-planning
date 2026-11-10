---
name: amd64-dev-image
code:
  - blizzard/.github/workflows/push.yml::dev-image
---

# The amd64-only dev image

Builds the dev image for amd64 only. Delivers `engineering:amd64-dev-image` and `product:arm64-releases`; builds
`spec/dev-image.md`. Independent of every other slice.

## Change

- `push.yml`'s `dev-image` builds `--platform linux/amd64` and drops `docker/setup-qemu-action`.
- `release.yml` is unchanged.
- `docs/ci.md` says the first arm64 build of any change is the release candidate's.
- The change's description records the job's time before and after (`engineering:gate-measured`).

## Acceptance

- `edge` and `sha-<sha>` are single-platform amd64 images, signed and attested as before.
- A release still publishes amd64 and arm64 images with a bill of materials per platform.
