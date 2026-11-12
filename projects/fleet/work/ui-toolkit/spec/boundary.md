# The domain boundary

The frontend's own domain, and the one place the wire is translated (`decision:ui-toolkit/frontend-domain`).

## System

```text
  generated client (fleet/src/lib/api/{hub,runner})          wire types: CadenceCreateRequest, Problem, …
        │
  *.query.ts · *.mutations.ts      data access: the only callers of the client (unchanged)
        │
  *.translate.ts                   wire ⇄ domain, refusal → domain field; the only other importers of wire types
        │
  *.domain.ts                      domain types and pure domain functions; no Angular, no wire
        │
  *.service.ts                     inject*Service(): domain in, domain out; composes data access and translators
        │
  container component  ──▶  presentational view  ──▶  kit
```

| Concept file | Imports wire types | Imports Angular | Pinned by |
| --- | --- | --- | --- |
| `<concept>.query.ts`, `<concept>.mutations.ts` | yes | yes | existing specs |
| `<concept>.translate.ts` | yes | no | `<concept>.translate.spec.ts`, no `TestBed` |
| `<concept>.domain.ts` | no | no | `<concept>.domain.spec.ts`, no `TestBed` |
| `<concept>.form.ts` (validation rules) | no | `@angular/forms/signals` only | `<concept>.form.spec.ts` |
| `<concept>.service.ts` | no | yes | `<concept>.service.spec.ts` |
| `*.model.ts`, components, views | no | per existing rules | existing specs |

A unit's files sit in its feature folder, as `*.query.ts` and `*.model.ts` do today, under the existing
`bzh:frontend-placement` and `bzh:frontend-package-layers` tables. A concept two features share goes down to the
project's kernel (`core/`), or to `fleet` where both apps reach it.

| blizzard-context | Change |
| --- | --- |
| `architecture/frontend-structure/wire.md` | `bzh:frontend-wire-conformist` replaced by `bzh:frontend-domain-boundary`, its pending-override exception reworded with it; `bzh:frontend-open-vocabulary` scoped to translators |
| `architecture/frontend-structure/containers.md` | `bzh:frontend-containers-compose`: a judgment once sent to the wire lives in a `*.domain.ts` |
| `architecture/frontend-structure/kit.md` | `bzh:frontend-kit-floor` dependency list (`decision:ui-toolkit/kit-dependencies`) |
| every other file that cites the old rule, in blizzard-context and in blizzard's own source and exemplars | the citation replaced |
| `verification/blizzard/commands/web/static-checks.md` | the boundary sweep's scope, report, and exemption list; the open-vocabulary sweep's narrowed scope |
| `exemplars/` web slice | gains the cadence concept's domain, translator, form, and service |

| `web/` gate | Change |
| --- | --- |
| Domain-boundary detector (new) | fails a file outside `*.query.ts`, `*.mutations.ts`, `*.translate.ts`, and `fleet/src/lib/api/` that imports from the generated `types.gen.ts`, judged by the import's source module, not its name, so a domain type may share a wire type's name; a unit on `DOMAIN_BOUNDARY_EXEMPT` passes |
| `DOMAIN_BOUNDARY_EXEMPT` (new) | every unit that reads a wire type outside the boundary when the sweep lands; a unit leaves it in the landing that moves it; empty at delivery |
| `wire-conformist-sweep.js` | renamed for the new rule; its client-call and backend-citation detectors are kept as they are |
| `open-vocabulary-sweep.js` | scoped to translators, where wire vocabularies are read |
| `eslint.config.js` | restricted imports, as `*.model.ts` has today: no Angular in `*.domain.ts` or `*.translate.ts`; only `@angular/forms/signals` in `*.form.ts`; `@angular/aria` only in the kit |

## Domain

```ts
// bzh:frontend-domain-boundary — a wire vocabulary becomes a closed domain union with an explicit unknown.
// The translator is the only place an unforeseen wire value is handled (bzh:frontend-open-vocabulary, now scoped here).
export type ChunkStatus =
  | 'not-ready' | 'ready' | 'running' | 'delivering' | 'waiting-on-human' | 'needs-human'
  | 'paused' | 'stopped' | 'done' | 'unknown';

// A judgment the wire already carries, such as a chunk's `terminal`, is translated, not re-derived. A judgment the
// frontend makes from what it holds is a pure domain function, pinned by its spec.
```

## Interfaces

```ts
// garden/core/cadences.query.ts — a read hands its consumers domain types; the cache keeps the wire response.
export function injectHubCadencesQuery() {
  const project = injectProjectFilter();
  return injectQuery(() => ({
    queryKey: lensedKey(hubCadencesKey, project()),
    queryFn: async (): Promise<CadenceView[]> => { /* as today */ },
    select: (wire: CadenceView[]): readonly Cadence[] => wire.map(fromCadenceView), // translator
  }));
}

// garden/core/cadence.translate.ts, beside garden/core/cadence.domain.ts: in the kernel, because the query in
// garden/core and more than one garden feature read them.
export function fromCadenceView(wire: CadenceView): Cadence;
export function toDraft(cadence: Cadence): CadenceDraft;
export function toCreateRequest(draft: CadenceDraft): CadenceCreateRequest;
export function toEditRequest(before: CadenceDraft, after: CadenceDraft): CadenceEditRequest; // sparse: changed only
export const CADENCE_REFUSALS: RefusalMap<CadenceDraft>; // spec/forms.md
```

A view-model file (`*.model.ts`) that derived from a wire view today takes the domain type instead. Its spec builds
domain literals, not wire literals, and `fleet/testing/builders` gains a domain builder beside each wire builder that a
translator spec still needs.
