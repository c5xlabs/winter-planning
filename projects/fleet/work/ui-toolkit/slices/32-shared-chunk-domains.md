---
name: shared-chunk-domains
external: github:paul-gross/blizzard#2286
depends-on:
  - shared-core-domains
code:
  - blizzard/web/projects/fleet/src/lib/chunk/chunk-detail/chunk-timeline.ts::ChunkTimeline
  - blizzard/web/projects/fleet/src/lib/transcripts/transcript-viewer.ts::TranscriptViewer
---

# Move shared chunk and transcript presentation onto domains

Migrates shared chunk detail, artifacts, timelines, and transcripts through [boundary.md](../spec/boundary.md#system)'s
domain boundary for `product:domain-boundary`.

## Change

- Replace generated input/model types with shared domain contracts and domain builders.
- Update constructing callers in both apps with each changed shared signature, without taking on their unrelated page
  logic.
- Keep artifact content, transcript streaming/ranges, actions, and layout behavior unchanged.

## Acceptance

- Translator/model tests preserve populated, empty, partial, and unknown-value cases for the shared presentation
  contracts.
- Both-app AOT compilation catches every changed input construction site; existing artifact/transcript tests still pass.
- `web:story-render` and affected browser layout cases remain green.
- `web:structural-gate` finds no generated-type import in moved components, their model specs, or stories; exemptions
  shrink.
